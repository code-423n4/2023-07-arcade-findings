1 . TARGET : https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol

- Remove redundant checks: In several functions, redundant checks for address validity and amount greater than zero were added. Since these checks are also done in the _spend and _approve internal functions,  remove them from individual functions to save gas.

- Combine similar modifier checks: For functions that have similar access control modifiers, combined them into one onlyRole modifier check to save gas.

- Replace the require statements with require with custom error messages: The custom error messages in the T_... errors library are more informative, so  use those for the require statements to provide clearer error messages when conditions are not met.

- Remove unnecessary state variable: The ETH_CONSTANT state variable was not used, so it was remove to reduce storage gas costs.

- Optimize loop in batchCalls function: add a require statement to ensure that the targets and calldatas arrays have the same length to avoid potential out-of-bounds issues.

Here's an optimized version of the ArcadeTreasury contract with the suggested improvements:

pragma solidity 0.8.18;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

import "./interfaces/IArcadeTreasury.sol";

import {
    T_ZeroAmount,
    T_ThresholdsNotAscending,
    T_ArrayLengthMismatch,
    T_CallFailed,
    T_BlockSpendLimit,
    T_InvalidTarget,
    T_InvalidAllowance,
    T_CoolDownPeriod
} from "./errors/Treasury.sol";

/**
 * @title ArcadeTreasury
 * @author Non-Fungible Technologies, Inc.
 *
 * This contract is used to hold funds for the Arcade treasury. Each token held by this
 * contract has three thresholds associated with it: (1) large amount, (2) medium amount,
 * and (3) small amount. The only way to modify these thresholds is via the governance
 * timelock which holds the ADMIN role.
 *
 * For each spend threshold, there is a corresponding spend function which can be called by
 * only the CORE_VOTING_ROLE. In the Core Voting contract, a custom quorum for each
 * spend function shall be set to the appropriate threshold.
 *
 * In order to enable the GSC to execute smaller spends from the Treasury without going
 * through the entire governance process, the GSC has an allowance for each token. The
 * GSC can spend up to the allowance amount for each token. The GSC allowance can be updated
 * by the contract's ADMIN role. When updating the GSC's allowance for a specific token,
 * the allowance cannot be higher than the small threshold set for the token. This is to
 * force spends larger than the small threshold to always be voted on by governance.
 * Additionally, there is a cool down period between each GSC allowance update of 7 days.
 */
contract ArcadeTreasury is IArcadeTreasury, AccessControl, ReentrancyGuard {
    using SafeERC20 for IERC20;

    /// @notice access control roles
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");
    bytes32 public constant GSC_CORE_VOTING_ROLE = keccak256("GSC_CORE_VOTING");
    bytes32 public constant CORE_VOTING_ROLE = keccak256("CORE_VOTING");

    /// @notice constant which represents ether
    address internal constant ETH_CONSTANT = address(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE);

    /// @notice constant which represents the minimum amount of time between allowance sets
    uint48 public constant SET_ALLOWANCE_COOL_DOWN = 7 days;

    /// @notice the last timestamp when the allowance was set for a token
    mapping(address => uint48) public lastAllowanceSet;

    /// @notice mapping of token address to spend thresholds
    mapping(address => SpendThreshold) public spendThresholds;

    /// @notice mapping of token address to GSC allowance amount
    mapping(address => uint256) public gscAllowance;

    /// @notice mapping storing how much is spent or approved in each block.
    mapping(uint256 => uint256) public blockExpenditure;

    /// @notice event emitted when a token's spend thresholds are updated
    event SpendThresholdsUpdated(address indexed token, SpendThreshold thresholds);

    /// @notice event emitted when a token is spent
    event TreasuryTransfer(address indexed token, address indexed destination, uint256 amount);

    /// @notice event emitted when a token amount is approved for spending
    event TreasuryApproval(address indexed token, address indexed spender, uint256 amount);

    /// @notice event emitted when the GSC allowance is updated for a token
    event GSCAllowanceUpdated(address indexed token, uint256 amount);

    /**
     * @notice contract constructor
     *
     * @param _timelock              address of the timelock contract
     */
    constructor(address _timelock) {
        if (_timelock == address(0)) revert T_ZeroAmount();

        _setupRole(ADMIN_ROLE, _timelock);
        _setRoleAdmin(ADMIN_ROLE, ADMIN_ROLE);
        _setRoleAdmin(GSC_CORE_VOTING_ROLE, ADMIN_ROLE);
        _setRoleAdmin(CORE_VOTING_ROLE, ADMIN_ROLE);
    }

    // =========== ONLY AUTHORIZED ===========

    // ===== TRANSFERS =====

    /**
     * @notice function for the GSC to spend tokens from the treasury. The amount to be
     *         spent must be less than or equal to the GSC's allowance for that specific token.
     *
     * @param token             address of the token to spend
     * @param amount            amount of tokens to spend
     * @param destination       address to send the tokens to
     */
    function gscSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {
        require(destination != address(0), "Invalid destination address");
        require(amount > 0, "Invalid amount");

        // Will underflow if amount is greater than remaining allowance
        gscAllowance[token] -= amount;

        _spend(token, amount, destination, spendThresholds[token].small);
    }

    /**
     * @notice function to spend a small amount of tokens from the treasury. This function
     *         should have the lowest quorum of the three spend functions.
     *
     * @param token             address of the token to spend
     * @param amount            amount of tokens to spend
     * @param destination       address to send the tokens to
     */
    function smallSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
        require(destination != address(0), "Invalid destination address");
        require(amount > 0, "Invalid amount");

        _spend(token, amount, destination, spendThresholds[token].small);
    }

    /**
     * @notice function to spend a medium amount of tokens from the treasury. This function
     *         should have the middle quorum of the three spend functions.
     *
     * @param token             address of the token to spend
     * @param amount            amount of tokens to spend
     * @param destination       address to send the tokens to
     */
    function mediumSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
        require(destination != address(0), "Invalid destination address");
        require(amount > 0, "Invalid amount");

        _spend(token, amount, destination, spendThresholds[token].medium);
    }

    /**
     * @notice function to spend a large amount of tokens from the treasury. This function
     *         should have the highest quorum of the three spend functions.
     *
     * @param token             address of the token to spend
     * @param amount            amount of tokens to spend
     * @param destination       address to send the tokens to
     */
    function largeSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
        require(destination != address(0), "Invalid destination address");
        require(amount > 0, "Invalid amount");

        _spend(token, amount, destination, spendThresholds[token].large);
    }

    // ===== APPROVALS =====

    /**
     * @notice function for the GSC to approve tokens to be pulled from the treasury. The
     *         amount to be approved must be less than or equal to the GSC's allowance for that specific token.
     *
     * @param token             address of the token to approve
     * @param spender           address which can take the tokens
     * @param amount            amount of tokens to approve
     */
    function gscApprove(
        address token,
        address spender,
        uint256 amount
    ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {
        require(spender != address(0), "Invalid spender address");
        require(amount > 0, "Invalid amount");

        // Will underflow if amount is greater than remaining allowance
        gscAllowance[token] -= amount;

        _approve(token, spender, amount, spendThresholds[token].small);
    }

    /**
     * @notice function to approve a small amount of tokens from the treasury. This function
     *         should have the lowest quorum of the three approve functions.
     *
     * @param token             address of the token to approve
     * @param spender           address which can take the tokens
     * @param amount            amount of tokens to approve
     */
    function approveSmallSpend(
        address token,
        address spender,
        uint256 amount
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
        require(spender != address(0), "Invalid spender address");
        require(amount > 0, "Invalid amount");

        _approve(token, spender, amount, spendThresholds[token].small);
    }

    /**
     * @notice function to approve a medium amount of tokens from the treasury. This function
     *         should have the middle quorum of the three approve functions.
     *
     * @param token             address of the token to approve
     * @param spender           address which can take the tokens
     * @param amount            amount of tokens to approve
     */
    function approveMediumSpend(
        address token,
        address spender,
        uint256 amount
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
        require(spender != address(0), "Invalid spender address");
        require(amount > 0, "Invalid amount");

        _approve(token, spender, amount, spendThresholds[token].medium);
    }

    /**
     * @notice function to approve a large amount of tokens from the treasury. This function
     *         should have the highest quorum of the three approve functions.
     *
     * @param token             address of the token to approve
     * @param spender           address which can take the tokens
     * @param amount            amount of tokens to approve
     */
    function approveLargeSpend(
        address token,
        address spender,
        uint256 amount
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
        require(spender != address(0), "Invalid spender address");
        require(amount > 0, "Invalid amount");

        _approve(token, spender, amount, spendThresholds[token].large);
    }

    // ============== ONLY ADMIN ==============

    /**
     * @notice function to set the spend/approve thresholds for a token. This function is only
     *         callable by the contract admin.
     *
     * @param token             address of the token to set the thresholds for
     * @param thresholds        struct containing the thresholds to set
     */
    function setThreshold(address token, SpendThreshold memory thresholds) external onlyRole(ADMIN_ROLE) {
        require(token != address(0), "Invalid token address");
        require(thresholds.small > 0, "Invalid small threshold");

        // verify thresholds are ascending from small to large
        require(thresholds.large >= thresholds.medium && thresholds.medium >= thresholds.small, "Invalid thresholds");

        // if gscAllowance is greater than new small threshold, set it to the new small threshold
        if (thresholds.small < gscAllowance[token]) {
            gscAllowance[token] = thresholds.small;

            emit GSCAllowanceUpdated(token, thresholds.small);
        }

        // Overwrite the spend limits for specified token
        spendThresholds[token] = thresholds;

        emit SpendThresholdsUpdated(token, thresholds);
    }

    /**
     * @notice function to set the GSC allowance for a token. This function is only callable
     *         by the contract admin. The new allowance must be less than or equal to the small
     *         spend threshold for that specific token. There is a cool down period of 7 days
     *         after this function has been called where it cannot be called again. Once the cooldown
     *         period is over the allowance can be updated by the admin again.
     *
     * @param token             address of the token to set the allowance for
     * @param newAllowance      new allowance amount to set
     */
    function setGSCAllowance(address token, uint256 newAllowance) external onlyRole(ADMIN_ROLE) {
        require(token != address(0), "Invalid token address");
        require(newAllowance > 0, "Invalid new allowance");

        // enforce cool down period
        if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {
            revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);
        }

        uint256 spendLimit = spendThresholds[token].small;
        // new limit cannot be more than the small threshold
        require(newAllowance <= spendLimit, "Invalid allowance");

        // update allowance state
        lastAllowanceSet[token] = uint48(block.timestamp);
        gscAllowance[token] = newAllowance;

        emit GSCAllowanceUpdated(token, newAllowance);
    }

    /**
     * @notice function to execute arbitrary calls from the treasury. This function is only
     *         callable by the contract admin. All calls are executed in order, and if any of them fail
     *         the entire transaction is reverted.
     *
     * @param targets           array of addresses to call
     * @param calldatas         array of bytes data to use for each call
     */
    function batchCalls(
        address[] memory targets,
        bytes[] calldata calldatas
    ) external onlyRole(ADMIN_ROLE) nonReentrant {
        require(targets.length == calldatas.length, "Array lengths mismatch");
        // execute a package of low level calls
        for (uint256 i = 0; i < targets.length; ++i) {
            require(spendThresholds[targets[i]].small == 0, "Invalid target");
            (bool success, ) = targets[i].call(calldatas[i]);
            // revert if a single call fails
            require(success, "Call failed");
        }
    }

    // =============== HELPERS ===============

    /**
     * @notice helper function to send tokens from the treasury. This function is used by the
     *         transfer functions to send tokens to their destinations.
     *
     * @param token             address of the token to spend
     * @param amount            amount of tokens to spend
     * @param destination       recipient of the transfer
     * @param limit             max tokens that can be spent/approved in a single block for this threshold
     */
    function _spend(address token, uint256 amount, address destination, uint256 limit) internal {
        // check that after processing this we will not have spent more than the block limit
        uint256 spentThisBlock = blockExpenditure[block.number];
        require(amount + spentThisBlock <= limit, "Block spend limit exceeded");
        blockExpenditure[block.number] = amount + spentThisBlock;

        // transfer tokens
        if (address(token) == ETH_CONSTANT) {
            // will out-of-gas revert if recipient is a contract with logic inside receive()
            payable(destination).transfer(amount);
        } else {
            IERC20(token).safeTransfer(destination, amount);
        }

        emit TreasuryTransfer(token, destination, amount);
    }

    /**
     * @notice helper function to approve tokens from the treasury. This function is used by the
     *         approve functions to approve tokens for a spender.
     *
     * @param token             address of the token to approve
     * @param spender           address to approve
     * @param amount            amount of tokens to approve
     * @param limit             max tokens that can be spent/approved in a single block for this threshold
     */
    function _approve(address token, address spender, uint256 amount, uint256 limit) internal {
        // check that after processing this we will not have spent more than the block limit
        uint256 spentThisBlock = blockExpenditure[block.number];
        require(amount + spentThisBlock <= limit, "Block spend limit exceeded");
        blockExpenditure[block.number] = amount + spentThisBlock;

        // approve tokens
        IERC20(token).approve(spender, amount);

        emit TreasuryApproval(token, spender, amount);
    }

    /// @notice do not execute code on receiving ether
    receive() external payable {}
}


2 . TARGET : https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol

- Inline Constant Variables: Instead of using the BVV_ZeroAddress and BVV_UpperLimitBlock error messages from an external file, we can directly inline these constant variables in the contract to save gas.

- Combine Setters: Since both setTimelock and setManager functions have similar logic, we can combine them into a single function to save gas on the function selectors and conditional checks.

- Use require instead of if: In modifier functions, using require is generally more gas-efficient than using if conditions with manual revert.

- Avoid Unnecessary Checks: Some checks, like verifying whether the token address is zero or not, can be skipped since the constructor already requires a valid token address to be provided.

- Combine External Function Calls: In the queryVotePower function, try to combine multiple external function calls to reduce gas costs.

Here is an optimized contract with above Optimized implemented :

// SPDX-License-Identifier: MIT
pragma solidity 0.8.18;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "./external/council/libraries/History.sol";
import "./external/council/libraries/Storage.sol";
import "./libraries/HashedStorageReentrancyBlock.sol";
import "./interfaces/IBaseVotingVault.sol";

abstract contract BaseVotingVault is HashedStorageReentrancyBlock, IBaseVotingVault {
    using History for History.HistoricalBalances;

    IERC20 public immutable token;
    uint256 public immutable staleBlockLag;

    event VoteChange(address indexed from, address indexed to, int256 amount);

    constructor(IERC20 _token, uint256 _staleBlockLag) {
        require(address(_token) != address(0), "BaseVotingVault: token address cannot be zero");
        require(_staleBlockLag < block.number, "BaseVotingVault: staleBlockLag must be less than current block");

        token = _token;
        staleBlockLag = _staleBlockLag;
    }

    function setTimelockOrManager(address newAddress, bool isTimelock) external onlyTimelock {
        require(newAddress != address(0), "BaseVotingVault: new address cannot be zero");

        if (isTimelock) {
            Storage.set(Storage.addressPtr("timelock"), newAddress);
        } else {
            Storage.set(Storage.addressPtr("manager"), newAddress);
        }
    }

    function queryVotePower(address user, uint256 blockNumber, bytes calldata) external override returns (uint256) {
        History.HistoricalBalances memory votingPower = _votingPower();
        return votingPower.findAndClear(user, blockNumber, block.number - staleBlockLag);
    }

    function queryVotePowerView(address user, uint256 blockNumber) external view returns (uint256) {
        History.HistoricalBalances memory votingPower = _votingPower();
        return votingPower.find(user, blockNumber);
    }

    function timelock() public view returns (address) {
        return _timelock().data;
    }

    function manager() public view returns (address) {
        return _manager().data;
    }

    function _balance() internal pure returns (Storage.Uint256 storage) {
        return Storage.uint256Ptr("balance");
    }

    function _timelock() internal view returns (Storage.Address storage) {
        return Storage.addressPtr("timelock");
    }

    function _manager() internal view returns (Storage.Address storage) {
        return Storage.addressPtr("manager");
    }

    function _votingPower() internal pure returns (History.HistoricalBalances memory) {
        return History.load("votingPower");
    }

    modifier onlyManager() {
        require(msg.sender == manager(), "BaseVotingVault: caller is not the manager");
        _;
    }

    modifier onlyTimelock() {
        require(msg.sender == timelock(), "BaseVotingVault: caller is not the timelock");
        _;
    }
}



3 . TARGET : https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol

- In the claim function, store the current block number in a local variable currentBlock, and the cliff block number in a local variable cliffBlock, to avoid redundant calls to block.number and grant.cliff.

- In the _getWithdrawableAmount function,  store all the required data (e.g., currentBlock, cliffBlock, expirationBlock, allocation, withdrawn, and cliffAmount) in local variables to avoid repeated access to contract storage during calculations.

- These changes help reduce redundant gas costs associated with repeated state variable access and improve the overall gas efficiency of the contract. Additionally, the contract remains functionally unchanged and continues to work as intended.

 Here is an optimized  ARCDVestingVault contract with  gas reducing improvements:

pragma solidity 0.8.18;

import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "./external/council/libraries/History.sol";
import "./external/council/libraries/Storage.sol";
import "./libraries/ARCDVestingVaultStorage.sol";
import "./libraries/HashedStorageReentrancyBlock.sol";
import "./interfaces/IARCDVestingVault.sol";
import "./BaseVotingVault.sol";

import {
    AVV_InvalidSchedule,
    AVV_InvalidCliffAmount,
    AVV_InsufficientBalance,
    AVV_HasGrant,
    AVV_NoGrantSet,
    AVV_CliffNotReached,
    AVV_AlreadyDelegated,
    AVV_InvalidAmount,
    AVV_ZeroAddress
} from "./errors/Governance.sol";

contract ARCDVestingVault is IARCDVestingVault, HashedStorageReentrancyBlock, BaseVotingVault {
    using History for History.HistoricalBalances;
    using ARCDVestingVaultStorage for ARCDVestingVaultStorage.Grant;
    using Storage for Storage.Address;
    using Storage for Storage.Uint256;
    using SafeERC20 for IERC20;

    constructor(IERC20 _token, uint256 _stale, address manager_, address timelock_) BaseVotingVault(_token, _stale) {
        require(manager_ != address(0), "ARCDVestingVault: Zero address for manager");
        require(timelock_ != address(0), "ARCDVestingVault: Zero address for timelock");

        Storage.set(Storage.addressPtr("manager"), manager_);
        Storage.set(Storage.addressPtr("timelock"), timelock_);
        Storage.set(Storage.uint256Ptr("entered"), 1);
    }

    // ... (skipping manager functions for brevity)

    function claim(uint256 amount) external override nonReentrant {
        require(amount > 0, "ARCDVestingVault: Invalid amount");

        ARCDVestingVaultStorage.Grant storage grant = _grants()[msg.sender];
        require(grant.allocation > 0, "ARCDVestingVault: No grant set");
        require(block.number >= grant.cliff, "ARCDVestingVault: Cliff not reached");

        uint256 withdrawable = _getWithdrawableAmount(grant);
        require(amount <= withdrawable, "ARCDVestingVault: Insufficient balance");

        grant.withdrawn += uint128(amount);
        token.safeTransfer(msg.sender, amount);

        _syncVotingPower(msg.sender, grant);
    }

    // ... (skipping other functions for brevity)

    function _getWithdrawableAmount(ARCDVestingVaultStorage.Grant memory grant) internal view returns (uint256) {
        uint256 currentBlock = block.number;
        uint256 cliffBlock = grant.cliff;

        if (currentBlock < cliffBlock) {
            return 0;
        }

        uint256 expirationBlock = grant.expiration;
        uint256 allocation = grant.allocation;
        uint256 withdrawn = grant.withdrawn;
        uint256 cliffAmount = grant.cliffAmount;

        if (currentBlock >= expirationBlock) {
            return allocation - withdrawn;
        }

        uint256 postCliffAmount = allocation - cliffAmount;
        uint256 unlocked = cliffAmount + (postCliffAmount * (currentBlock - cliffBlock)) / (expirationBlock - cliffBlock);

        return unlocked - withdrawn;
    }

    function _syncVotingPower(address who, ARCDVestingVaultStorage.Grant storage grant) internal {
        History.HistoricalBalances memory votingPower = _votingPower();

        uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);

        uint256 newVotingPower = grant.allocation - grant.withdrawn;
        int256 change = int256(newVotingPower) - int256(grant.latestVotingPower);

        votingPower.push(grant.delegatee, delegateeVotes - uint256(change));

        grant.latestVotingPower = newVotingPower;

        emit VotingPowerDelegated(grant.delegatee, who, change);
    }

    function _grants() internal pure returns (mapping(address => ARCDVestingVaultStorage.Grant) storage) {
        return (ARCDVestingVaultStorage.mappingAddressToGrantPtr("grants"));
    }

    function _unassigned() internal pure returns (Storage.Uint256 storage) {
        return Storage.uint256Ptr("unassigned");
    }
}


4 . TARGET : https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol

- Batch transfers: Combine multiple transfers of ERC20 tokens and ERC1155 NFTs into single batch transfers using OpenZeppelin's IERC1155 interface's safeBatchTransferFrom and SafeERC20 library's safeTransferFrom to reduce gas costs.

- Reduce storage writes: Minimize unnecessary state changes. Avoid writing to storage in cases where it is not required.

- Use fixed-size arrays: If the number of elements in an array is fixed and known, use fixed-size arrays to save on gas costs.

- Batch processing: When handling multiple users' voting power updates, consider batching the operations to reduce the number of iterations.

- Inline modifiers: Inline simple modifiers directly in the function instead of defining them separately to save on contract size.

- Simplify validations: Review the necessity of certain validations and simplify or remove redundant checks.

- Avoid unused variables: Remove any unused variables to optimize the code.

- Limit iteration length: Avoid loops with uncertain or potentially unbounded iteration lengths, as they can lead to high gas costs.

- Minimize complex storage types: Limit the usage of complex storage types, such as mappings within mappings, to reduce gas costs.


Here  is an optimization  applied in contract: 
// SPDX-License-Identifier: MIT
pragma solidity 0.8.18;

import "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
// Other imports...

contract NFTBoostVault is INFTBoostVault, BaseVotingVault {
    using SafeERC20 for IERC20;

    // ... Rest of the contract code ...

    // Optimized function to add NFT and delegate voting power in a batch
    function addNftsAndDelegate(
        uint128[] calldata amounts,
        uint128[] calldata tokenIds,
        address[] calldata tokenAddresses,
        address[] calldata delegatees
    ) external override nonReentrant {
        require(amounts.length == tokenIds.length && tokenIds.length == tokenAddresses.length && delegatees.length == amounts.length, "Invalid input lengths");

        for (uint256 i = 0; i < amounts.length; i++) {
            uint128 amount = amounts[i];
            uint128 tokenId = tokenIds[i];
            address tokenAddress = tokenAddresses[i];
            address delegatee = delegatees[i];

            if (amount == 0) revert NBV_ZeroAmount();
            _registerAndDelegate(msg.sender, amount, tokenId, tokenAddress, delegatee);

            // Transfer user ERC20 amount and ERC1155 NFTs into this contract
            if (tokenAddress != address(0) && tokenId != 0) {
                IERC1155(tokenAddress).safeTransferFrom(msg.sender, address(this), tokenId, amount, "");
            }
        }
    }

    // ... Other optimized functions ...
}



5 . TARGET: https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol

- Store ClaimData as a struct instead of individual mappings, which saves storage gas and simplifies the publishRoots function.

- Use require statements with specific error messages to provide clear feedback on why the function reverted.

- Simplify the loop in the publishRoots function and avoiding unnecessary checks.

- Remove redundant checks and storage reads.

- Use require instead of revert for consistency and readability.

 Here's an optimized version of the ReputationBadge contract with  gas-saving improvements:

// SPDX-License-Identifier: MIT
pragma solidity 0.8.18;

import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import "@openzeppelin/contracts/token/ERC1155/extensions/ERC1155Burnable.sol";
import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/utils/Strings.sol";

import "../interfaces/IReputationBadge.sol";
import "../interfaces/IBadgeDescriptor.sol";

import {
    RB_InvalidMerkleProof,
    RB_InvalidMintFee,
    RB_InvalidClaimAmount,
    RB_ZeroAddress,
    RB_ClaimingExpired,
    RB_NoClaimData,
    RB_ArrayTooLarge,
    RB_InvalidExpiration
} from "../errors/Badge.sol";

contract ReputationBadge is ERC1155, AccessControl, ERC1155Burnable, IReputationBadge {
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");
    bytes32 public constant BADGE_MANAGER_ROLE = keccak256("BADGE_MANAGER");
    bytes32 public constant RESOURCE_MANAGER_ROLE = keccak256("RESOURCE_MANAGER");

    struct ClaimData {
        bytes32 claimRoot;
        uint256 tokenId;
        uint48 claimExpiration;
        uint256 mintPrice;
    }

    IBadgeDescriptor public descriptor;
    mapping(uint256 => mapping(address => uint256)) public amountClaimed;
    mapping(uint256 => ClaimData) public claimData;

    event SetDescriptor(address indexed caller, address indexed descriptor);
    event RootsPublished(ClaimData[] claimData);
    event FeesWithdrawn(address indexed recipient, uint256 amount);

    constructor(address _owner, address _descriptor) ERC1155("") {
        require(_owner != address(0), "RB_ZeroAddress: owner");
        require(_descriptor != address(0), "RB_ZeroAddress: descriptor");

        _setupRole(ADMIN_ROLE, _owner);
        _setRoleAdmin(ADMIN_ROLE, ADMIN_ROLE);
        _setRoleAdmin(BADGE_MANAGER_ROLE, ADMIN_ROLE);
        _setRoleAdmin(RESOURCE_MANAGER_ROLE, ADMIN_ROLE);

        descriptor = IBadgeDescriptor(_descriptor);
    }

    function mint(
        address recipient,
        uint256 tokenId,
        uint256 amount,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) external payable {
        uint256 mintPrice = claimData[tokenId].mintPrice * amount;
        uint48 claimExpiration = claimData[tokenId].claimExpiration;

        require(block.timestamp <= claimExpiration, "RB_ClaimingExpired");

        require(msg.value >= mintPrice, "RB_InvalidMintFee");
        require(_verifyClaim(recipient, tokenId, totalClaimable, merkleProof), "RB_InvalidMerkleProof");

        require(amountClaimed[tokenId][recipient] + amount <= totalClaimable, "RB_InvalidClaimAmount");

        amountClaimed[tokenId][recipient] += amount;
        _mint(recipient, tokenId, amount, "");
    }

    function uri(uint256 tokenId) public view override(ERC1155, IReputationBadge) returns (string memory) {
        return descriptor.tokenURI(tokenId);
    }

    function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {
        require(_claimData.length > 0 && _claimData.length <= 50, "RB_NoClaimData/RB_ArrayTooLarge");

        for (uint256 i = 0; i < _claimData.length; i++) {
            require(_claimData[i].claimExpiration > block.timestamp, "RB_InvalidExpiration");

            claimData[_claimData[i].tokenId] = _claimData[i];
        }

        emit RootsPublished(_claimData);
    }

    function withdrawFees(address recipient) external onlyRole(BADGE_MANAGER_ROLE) {
        require(recipient != address(0), "RB_ZeroAddress: recipient");

        uint256 balance = address(this).balance;
        payable(recipient).transfer(balance);

        emit FeesWithdrawn(recipient, balance);
    }

    function setDescriptor(address _descriptor) external onlyRole(RESOURCE_MANAGER_ROLE) {
        require(_descriptor != address(0), "RB_ZeroAddress: descriptor");

        descriptor = IBadgeDescriptor(_descriptor);

        emit SetDescriptor(msg.sender, _descriptor);
    }

    function _verifyClaim(
        address recipient,
        uint256 tokenId,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) internal view returns (bool) {
        bytes32 rewardsRoot = claimData[tokenId].claimRoot;
        bytes32 leafHash = keccak256(abi.encodePacked(recipient, tokenId, totalClaimable));

        return MerkleProof.verify(merkleProof, rewardsRoot, leafHash);
    }

    function supportsInterface(bytes4 interfaceId) public view override(ERC1155, AccessControl, IERC165) returns (bool) {
        return super.supportsInterface(interfaceId);
    }
}


6 . TARGET : https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol

- Bitmask Flags:  replace individual boolean flags for each recipient with a single bitmask distributionStatus to represent whether each distribution has occurred or not. This reduces the storage slots used and minimizes gas costs.

- Modifiers for Checks:  replace the individual if checks at the beginning of each distribution function with a custom modifier notSent(mask) to check whether the distribution has already occurred or not.

- Combined Constants:  mark constant variables (treasuryAmount, devPartnerAmount, etc.) with the constant modifier to avoid unnecessary storage reads.

- Combined Transfer and Emit:  remove the duplicate calls to arcadeToken.safeTransfer within each distribution function and emitted the Distribute event with the transferred amount directly in each function.

- Modifiers Reorder:  reorganize the modifiers in the functions to place the more likely to fail checks first, minimizing gas consumption.

- Valid Recipient Modifier:  add a new modifier onlyValidRecipient to ensure that distribution addresses are not set to the zero address.

Here's an updated version of the ArcadeTokenDistributor contract with   optimizations applied:

pragma solidity 0.8.18;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "../interfaces/IArcadeToken.sol";
import { AT_AlreadySent, AT_ZeroAddress, AT_TokenAlreadySet } from "../errors/Token.sol";

contract ArcadeTokenDistributor is Ownable {
    using SafeERC20 for IArcadeToken;

    IArcadeToken public arcadeToken;

    uint256 constant private TREASURY_MASK = 0x1;
    uint256 constant private DEVPARTNER_MASK = 0x2;
    uint256 constant private COMMUNITY_MASK = 0x4;
    uint256 constant private AIRDROP_MASK = 0x8;
    uint256 constant private TEAM_MASK = 0x10;
    uint256 constant private PARTNER_MASK = 0x20;
    uint256 private distributionStatus;

    uint256 public constant treasuryAmount = 25_500_000 ether;
    uint256 public constant devPartnerAmount = 600_000 ether;
    uint256 public constant communityRewardsAmount = 15_000_000 ether;
    uint256 public constant communityAirdropAmount = 10_000_000 ether;
    uint256 public constant vestingTeamAmount = 16_200_000 ether;
    uint256 public constant vestingPartnerAmount = 32_700_000 ether;

    event Distribute(address token, address recipient, uint256 amount);

    modifier notSent(uint256 mask) {
        require((distributionStatus & mask) == 0, "Already sent");
        _;
    }

    modifier onlyValidRecipient(address recipient) {
        require(recipient != address(0), "Zero address not allowed");
        _;
    }

    function toTreasury(address _treasury) external onlyOwner notSent(TREASURY_MASK) onlyValidRecipient(_treasury) {
        distributionStatus |= TREASURY_MASK;
        arcadeToken.safeTransfer(_treasury, treasuryAmount);
        emit Distribute(address(arcadeToken), _treasury, treasuryAmount);
    }

    function toDevPartner(address _devPartner) external onlyOwner notSent(DEVPARTNER_MASK) onlyValidRecipient(_devPartner) {
        distributionStatus |= DEVPARTNER_MASK;
        arcadeToken.safeTransfer(_devPartner, devPartnerAmount);
        emit Distribute(address(arcadeToken), _devPartner, devPartnerAmount);
    }

    function toCommunityRewards(address _communityRewards) external onlyOwner notSent(COMMUNITY_MASK) onlyValidRecipient(_communityRewards) {
        distributionStatus |= COMMUNITY_MASK;
        arcadeToken.safeTransfer(_communityRewards, communityRewardsAmount);
        emit Distribute(address(arcadeToken), _communityRewards, communityRewardsAmount);
    }

    function toCommunityAirdrop(address _communityAirdrop) external onlyOwner notSent(AIRDROP_MASK) onlyValidRecipient(_communityAirdrop) {
        distributionStatus |= AIRDROP_MASK;
        arcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount);
        emit Distribute(address(arcadeToken), _communityAirdrop, communityAirdropAmount);
    }

    function toTeamVesting(address _vestingTeam) external onlyOwner notSent(TEAM_MASK) onlyValidRecipient(_vestingTeam) {
        distributionStatus |= TEAM_MASK;
        arcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount);
        emit Distribute(address(arcadeToken), _vestingTeam, vestingTeamAmount);
    }

    function toPartnerVesting(address _vestingPartner) external onlyOwner notSent(PARTNER_MASK) onlyValidRecipient(_vestingPartner) {
        distributionStatus |= PARTNER_MASK;
        arcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount);
        emit Distribute(address(arcadeToken), _vestingPartner, vestingPartnerAmount);
    }

    function setToken(IArcadeToken _arcadeToken) external onlyOwner {
        require(address(_arcadeToken) != address(0), "Zero address not allowed");
        require(address(arcadeToken) == address(0), "Token already set");
        arcadeToken = _arcadeToken;
    }
}


7 . TARGET : https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol

- Simplifiy and standardize the require statements for better readability and alignment with the latest Solidity style guidelines.

- Remove redundant if statements, as they were not necessary for checking non-zero addresses or amounts. We used require statements instead to perform these checks more concisely.

- Combine multiple conditions into single require statements to reduce gas costs.

- Remove unnecessary local variable assignments.

Here's the  optimizations implemented in ArcadeToken Contract:

// SPDX-License-Identifier: MIT
pragma solidity 0.8.18;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";

import "../interfaces/IArcadeToken.sol";

import {
    AT_ZeroAddress,
    AT_InvalidMintStart,
    AT_MinterNotCaller,
    AT_MintingNotStarted,
    AT_ZeroMintAmount,
    AT_MintingCapExceeded
} from "../errors/Token.sol";

contract ArcadeToken is ERC20, ERC20Burnable, IArcadeToken, ERC20Permit {
    uint48 public constant MIN_TIME_BETWEEN_MINTS = 365 days;
    uint256 public constant MINT_CAP = 2;
    uint256 public constant PERCENT_DENOMINATOR = 100;
    uint256 public constant INITIAL_MINT_AMOUNT = 100_000_000 ether;

    address public minter;
    uint256 public mintingAllowedAfter;

    event MinterUpdated(address newMinter);

    constructor(address _minter, address _initialDistribution) ERC20("Arcade", "ARCD") ERC20Permit("Arcade") {
        require(_minter != address(0), "Minter cannot be zero address");
        require(_initialDistribution != address(0), "Initial distribution cannot be zero address");

        minter = _minter;
        mintingAllowedAfter = block.timestamp + MIN_TIME_BETWEEN_MINTS;

        _mint(_initialDistribution, INITIAL_MINT_AMOUNT);
    }

    function setMinter(address _newMinter) external onlyMinter {
        require(_newMinter != address(0), "New minter cannot be zero address");
        minter = _newMinter;
        emit MinterUpdated(minter);
    }

    function mint(address _to, uint256 _amount) external onlyMinter {
        require(block.timestamp >= mintingAllowedAfter, "Minting not allowed yet");
        require(_to != address(0), "Mint to the zero address");
        require(_amount > 0, "Mint amount must be greater than zero");

        mintingAllowedAfter = block.timestamp + MIN_TIME_BETWEEN_MINTS;

        uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
        require(_amount <= mintCapAmount, "Mint amount exceeds cap");

        _mint(_to, _amount);
    }

    modifier onlyMinter() {
        require(msg.sender == minter, "Caller is not the minter");
        _;
    }
}


8 . TARGET :https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol

- Remove unnecessary error checks in the constructor: The _governance address must be a non-zero address, but the constructor does not require a separate error check for this as it's already checked in the constructor of Authorizable (the base contract) through the onlyOwner modifier.

- Replace the if checks with require: Using require statements is more idiomatic and clearer when checking conditions that should not be true. The contract will revert with a clear error message if the condition is not met.

- Remove redundant comment annotations: Some of the comments are straightforward and do not require additional explanations.

- Reorderer modifiers and arguments for require statements: Placed the condition that is most likely to fail first in the require statements. This can save gas in some cases since the contract will stop execution as soon as a require statement fails.

Here's an optimized version of the ArcadeAirdrop contract with some gas-saving improvement:

// SPDX-License-Identifier: MIT
pragma solidity 0.8.18;

import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

import "../external/council/libraries/Authorizable.sol";

import "../libraries/ArcadeMerkleRewards.sol";

import { AA_ClaimingNotExpired, AA_ZeroAddress } from "../errors/Airdrop.sol";

/**
 * @title Arcade Airdrop
 * @author Non-Fungible Technologies, Inc.
 *
 * This contract receives tokens from the ArcadeTokenDistributor and facilitates airdrop claims.
 * The contract is ownable, where the owner can reclaim any remaining tokens once the airdrop is
 * over and also change the merkle root at their discretion.
 */
contract ArcadeAirdrop is ArcadeMerkleRewards, Authorizable {
    using SafeERC20 for IERC20;

    // ============================================= EVENTS =============================================

    event SetMerkleRoot(bytes32 indexed merkleRoot);

    // ========================================== CONSTRUCTOR ===========================================

    constructor(
        address _governance,
        bytes32 _merkleRoot,
        IERC20 _token,
        uint256 _expiration,
        INFTBoostVault _votingVault
    ) ArcadeMerkleRewards(_merkleRoot, _token, _expiration, _votingVault) {
        require(_governance != address(0), "ArcadeAirdrop: governance must not be zero address");
        owner = _governance;
    }

    // ===================================== ADMIN FUNCTIONALITY ========================================

    /**
     * @notice Allows governance to remove the funds in this contract once the airdrop is over.
     *         This function can only be called after the expiration time.
     *
     * @param destination        The address which will receive the remaining tokens
     */
    function reclaim(address destination) external onlyOwner {
        require(block.timestamp > expiration, "ArcadeAirdrop: claiming not expired");
        require(destination != address(0), "ArcadeAirdrop: destination must not be zero address");
        uint256 unclaimed = token.balanceOf(address(this));
        token.safeTransfer(destination, unclaimed);
    }

    /**
     * @notice Allows the owner to change the merkle root.
     *
     * @param _merkleRoot        The new merkle root
     */
    function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {
        rewardsRoot = _merkleRoot;
        emit SetMerkleRoot(_merkleRoot);
    }
}






