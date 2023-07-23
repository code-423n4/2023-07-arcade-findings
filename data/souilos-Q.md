# VULN 1 

## [LOW] Keccak Constant values should used to immutable rather than constant
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 48 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");


Found in line 49 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

    bytes32 public constant GSC_CORE_VOTING_ROLE = keccak256("GSC_CORE_VOTING");


Found in line 50 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

    bytes32 public constant CORE_VOTING_ROLE = keccak256("CORE_VOTING");


Found in line 44 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");


Found in line 45 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

    bytes32 public constant BADGE_MANAGER_ROLE = keccak256("BADGE_MANAGER");


Found in line 46 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

    bytes32 public constant RESOURCE_MANAGER_ROLE = keccak256("RESOURCE_MANAGER");

------------------------------------------------------------------------ 

### Mitigation 

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts. While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.










# VULN 2 

## [LOW] Empty receive()/payable fallback() function does not authenticate requests
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 397 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

    receive() external payable {}

------------------------------------------------------------------------ 

### Mitigation 

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))).Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas. Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.










# VULN 3 

## [LOW] Keccak Constant values should used to immutable rather than constant
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 48 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");


Found in line 49 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

    bytes32 public constant GSC_CORE_VOTING_ROLE = keccak256("GSC_CORE_VOTING");


Found in line 50 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

    bytes32 public constant CORE_VOTING_ROLE = keccak256("CORE_VOTING");


Found in line 44 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");


Found in line 45 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

    bytes32 public constant BADGE_MANAGER_ROLE = keccak256("BADGE_MANAGER");


Found in line 46 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

    bytes32 public constant RESOURCE_MANAGER_ROLE = keccak256("RESOURCE_MANAGER");

------------------------------------------------------------------------ 

### Mitigation 

By using immutable variables, the Keccak256 hash is computed at contract deployment time rather than at compilation time. This means that the value can be updated if the algorithm changes in a future compiler version, without breaking backward compatibility. Additionally, it provides better gas optimization, as the Keccak256 hash is computed only once at contract deployment instead of every time the variable is accessed during execution.










# VULN 4 

## [LOW] Use .call instead of .transfer to send ether
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 367 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

            payable(destination).transfer(amount);


Found in line 171 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        payable(recipient).transfer(balance);

------------------------------------------------------------------------ 

### Mitigation 

.transfer will relay 2300 gas and .call will relay all the gas. If the receive/fallback function from the recipient proxy contract has complex logic, using .transfer will fail, causing integration issues.Replace .transfer with .call. Note that the result of .call need to be checked.










# VULN 5 

## [LOW] Use the safe variant and ERC721.mint
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 122 at 2023-07-arcadexyz/contracts/token/ArcadeToken.sol:

        _mint(_initialDistribution, INITIAL_MINT_AMOUNT);


Found in line 159 at 2023-07-arcadexyz/contracts/token/ArcadeToken.sol:

        _mint(_to, _amount);


Found in line 119 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        _mint(recipient, tokenId, amount, "");

------------------------------------------------------------------------ 

### Mitigation 

.mint won’t check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset. OpenZeppelin recommendation is to use the safe variant of _mint. Replace _mint() with _safeMint().










# VULN 6 

## [LOW] Immutables should be in uppercase
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 33 at 2023-07-arcadexyz/contracts/BaseVotingVault.sol:

    IERC20 public immutable token;


Found in line 36 at 2023-07-arcadexyz/contracts/BaseVotingVault.sol:

    uint256 public immutable staleBlockLag;

------------------------------------------------------------------------ 

### Mitigation 

Immutables should be in uppercase, it helps to distinguish immutables from other types of variables and provides better code readability.
