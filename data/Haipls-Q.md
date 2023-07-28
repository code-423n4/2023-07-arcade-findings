## [01] Duplication in Contract Inheritance
When a contract inherits the same base contracts twice, it can add unnecessary complexity when analyzing the code structure. This happens when a contract directly inherits a base contract and also inherits another contract that already inherits this base contract.

### PROOF OF CONCEPT
Instances:
* [nft/ReputationBadge.sol#L39](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L39)
* [token/ArcadeToken.sol#L74](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L74)
```c
File: contracts/nft/ReputationBadge.sol

39: contract ReputationBadge is ERC1155, AccessControl, ERC1155Burnable, IReputationBadge { //@audit ERC1155Burnable also inherit ERC1155

------------------------------------------------------------------
File: contracts/token/ArcadeToken.sol

74: contract ArcadeToken is ERC20, ERC20Burnable, IArcadeToken, ERC20Permit { //@audit ERC20Burnable and ERC20Permit also inherit ERC20

```

### Mittigation 
To avoid double inheritance, it is recommended to review the contract's inheritance structure and remove unnecessary direct inheritance. If the contract already inherits the necessary properties and methods through another contract, additional direct inheritance is not needed.

## [02] Unnecessary Override of `DEFAULT_ADMIN_ROLE` to `ADMIN_ROLE`
By default, AccessControl from OpenZeppelin already includes the constant DEFAULT_ADMIN_ROLE defined as follows: 
```js
File: contracts/access/AccessControl.sol

56:    bytes32 public constant DEFAULT_ADMIN_ROLE = 0x00;
```

However, some contracts create their own ADMIN role that duplicates the functionality of DEFAULT_ADMIN_ROLE. Which is an overcomplication, it also creates an unnecessary public constant in the code

### PROOF OF CONCEPT
Instances:
* [nft/ReputationBadge.sol#L44](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L44)
* [ArcadeTrasury.sol#L48](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L48)

```js
File: contracts/nft/ReputationBadge.sol

44: bytes32 public constant ADMIN_ROLE = keccak256("ADMIN"); 

75: constructor(address _owner, address _descriptor) ERC1155("") {
    ...
        _setupRole(ADMIN_ROLE, _owner);
        _setRoleAdmin(ADMIN_ROLE, ADMIN_ROLE);
        _setRoleAdmin(BADGE_MANAGER_ROLE, ADMIN_ROLE);
        _setRoleAdmin(RESOURCE_MANAGER_ROLE, ADMIN_ROLE);
    ...
    }

------------------------------------------------------------------
File: contracts/ArcadeTreasury.sol

48:    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");

87:    constructor(address _timelock) {
        ...
            _setupRole(ADMIN_ROLE, _timelock);
            _setRoleAdmin(ADMIN_ROLE, ADMIN_ROLE);
            _setRoleAdmin(GSC_CORE_VOTING_ROLE, ADMIN_ROLE);
            _setRoleAdmin(CORE_VOTING_ROLE, ADMIN_ROLE);
        }
```

### Mittigation 
It's recommended to use the provided default mechanism if it fulfills the same purpose. This will reduce code duplication and improve code clarity. If the default role provided by AccessControl serves the required purpose, there is no need to define a new identical role.

## [03] Lack of 'amount > 0' Check, Allowing Potential Misuse of mint() Function
In the current implementation, there's a lack of validation check for the variable amount being greater than zero. This allows the mint() function to be called without minting any tokens, causing unnecessary event emissions that could mislead observers.

### PROOF OF CONCEPT
Instances:
* [nft/ReputationBadge.sol#L105](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L105)
```js
File: contracts/nft/ReputationBadge.sol

98: function mint(
        address recipient,
        uint256 tokenId,
        uint256 amount,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) external payable {
105:     uint256 mintPrice = mintPrices[tokenId] * amount; // @audit miss amount validation
```

### Mittigation 
To avoid confusion and wasted costs, should add a check at the start of the mint() function to make sure the amount being minted is more than zero.


## [04] `mint()` allows to accept more native currency than needed
The `mint()` function in the `ReputationBadge` contract currently doesn't restrict the number of native coins sent to it. This could potentially lead to scenarios where more funds are sent than are actually required by the function. Which will lead to user losses

### PROOF OF CONCEPT
Instances:
* [nft/ReputationBadge.sol#L109](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L109)
```js
File: contracts/nft/ReputationBadge.sol

109:         if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value); // @audit

```

### Mittigation 
Add a check to ensure msg.value is exactly equal to mintPrice. This will prevent overpayment scenarios:
```js
    if (msg.value != mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);
```


## [05] Absence of `merkleProof` Length Check
The contract uses OpenZeppelin's implementation of Merkle Proof validation. In the current setup, if an empty `merkleProof` array is passed, the validation simply compares `rewardsRoot` with `leafHash`, which may yield unexpected results and increases opportunities for manipulation

### PROOF OF CONCEPT

Instances:
* [nft/ReputationBadge.sol#L204](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L204)
```js
File: contracts/nft/ReputationBadge.sol

98: function mint(
        address recipient,
        uint256 tokenId,
        uint256 amount,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) external payable {
        uint256 mintPrice = mintPrices[tokenId] * amount; 
        uint48 claimExpiration = claimExpirations[tokenId];

        if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));
        if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);
        if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof(); // @audit

204:    function _verifyClaim(
            address recipient,
            uint256 tokenId,
            uint256 totalClaimable,
            bytes32[] calldata merkleProof // @audit miss min length check
        ) internal view returns (bool) {
            bytes32 rewardsRoot = claimRoots[tokenId];
            bytes32 leafHash = keccak256(abi.encodePacked(recipient, tokenId, totalClaimable));

            return MerkleProof.verify(merkleProof, rewardsRoot, leafHash); // @audit
        }
```

### Mittigation 
Add a check to ensure that the length of `merkleProof` is greater than 0 before the call to `MerkleProof.verify()`.


## [06] Call `ReputationBadge.publishRoots()` can be front-running
The call to publishRoots sets available tokens for purchase or modifies this list/recipients, etc. This can lead to situations where a user might front-run this call to gain an advantage. 


For example:

* A user is allocated 10 tokens with id 100 for purchase.
* The user negotiated to have their allocation changed to 20 tokens with id 200.
* The user front-run the `publishRoots` call, purchasing 10 tokens with id 100 before the call and 20 tokens with id 200 after the call.

### PROOF OF CONCEPT

Instances:
* [nft/ReputationBadge.sol#L140](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L140)
```js
File: contracts/nft/ReputationBadge.sol

140: function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {

```

### Mittigation 
A good approach to mitigate the issue can be to implement an additional function which revokes a user's right to mint tokens by resetting the claim root, claim expiration, and mint price for the user's token ID  before the new root for the tokenID is set


## [07] Call `ArcadeAirdrop.setMerkleRoot()` can be front-running
In the `ArcadeAirdrop` contract, the owner has the right to reassign `merkleRoot` as he wishes for situations he deems necessary: ​​change recipients, increase airdrop amounts, decrease, etc. Which could potentially be points of interest for malicious users who might see an advantage in changing merkleRoot and use it to gain an undue advantage.

Example situation:
* The owner wants to transfer the unused airdrop token amount to other addresses provided by users, users monitor the change and receive x2 tokens as received at the old address before the change of `merkleRoot` by front-running, and at the new address after the change of `merkleRoot`

### PROOF OF CONCEPT

Instances:
* [token/ArcadeAirdrop.sol#L75](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeAirdrop.sol#L75)
```js
File: contracts/token/ArcadeAirdrop.sol

75:  function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {
        rewardsRoot = _merkleRoot;

        emit SetMerkleRoot(_merkleRoot);
    }
```

### Mittigation 
One of the options is a description of the process that the owner will follow when changing `merkleRoot`, first nullifying it, viewing transactions that occurred before the change. Making changes as needed to the new merkleRoot, issuing a new correct `merkleRoot`


## [08] Checking the success of the call while ignoring the returned data is not always a guarantee of a successful call result
Since there are different contracts with their implementations, these implementations can include a situation when a transaction is unsuccessful, but this does not lead to revert

The most common example is the ERC20 standard, where the transfer/transferFrom etc transfer methods return a bool, which, according to the standard, indicates success. In this case, passing the transaction is not a guarantee of achieving the planned result, such as the transfer of 100 tokens, the transaction went through, but the tokens were not transferred.

### PROOF OF CONCEPT

Instances:
* [ArcadeTrasury.sol#L341](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L341)
```js
File: contracts/ArcadeTreasury.sol

333:function batchCalls(
        address[] memory targets,
        bytes[] calldata calldatas
    ) external onlyRole(ADMIN_ROLE) nonReentrant {
        if (targets.length != calldatas.length) revert T_ArrayLengthMismatch();
        // execute a package of low level calls
        for (uint256 i = 0; i < targets.length; ++i) {
            if (spendThresholds[targets[i]].small != 0) revert T_InvalidTarget(targets[i]);
            (bool success, ) = targets[i].call(calldatas[i]); // @audit
            // revert if a single call fails
            if (!success) revert T_CallFailed();
        }
    }
```

### Mittigation 
Check returnedData in case of bool check for true


## [09] Use increaseAllowance/decreaseAllowance instead of approve() to prevent front-running
Direct calls to approve may lead to possible issues such as front-running or "approve zero first", potentially leading to detrimental consequences.

### PROOF OF CONCEPT

Instances:
* [ArcadeTrasury.sol#L391](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L391)
```js
File: contracts/ArcadeTreasury.sol

        blockExpenditure[block.number] = amount + spentThisBlock;

        // approve tokens
391:    IERC20(token).approve(spender, amount);

        emit TreasuryApproval(token, spender, amount);
```

### Mittigation 
Use increaseAllowance/decreaseAllowance instead of approve()


## [10] batchCalls is not suitable for calling payable functions or transferring eth
`batchCalls` allows the `ADMIN_ROLE` to bypass any other methods and transfer tokens directly, neglecting any restrictions. However, it does not allow interaction with payable functions or the transfer of eth during the call, which is illogical. This leads to illogical limitations, which may lead to situations where it is necessary to do so but was not anticipated.

It may also be intentionally designed to limit interaction with ETH while enabling the transfer of tokens, but this is illogical. If that's the case, documentation should be added to explain this.

### PROOF OF CONCEPT
Instances:
* [ArcadeTrasury.sol#L333](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L333)
```js
File: contracts/ArcadeTreasury.sol

333:function batchCalls(
        address[] memory targets,
        bytes[] calldata calldatas
    ) external onlyRole(ADMIN_ROLE) nonReentrant { // @audit miss payable modifier
        if (targets.length != calldatas.length) revert T_ArrayLengthMismatch();
        // execute a package of low level calls
        for (uint256 i = 0; i < targets.length; ++i) {
            if (spendThresholds[targets[i]].small != 0) revert T_InvalidTarget(targets[i]);
            (bool success, ) = targets[i].call(calldatas[i]); // @audit miss {value}
            // revert if a single call fails
            if (!success) revert T_CallFailed();
        }
    }
```

### Mittigation 
Explicitly add the ability to transfer ETH and interact with payable functions, or clearly state in the comments that ETH interaction is limited, but allowed with any tokens.

## [11] ArcadeTresuary forbids zeroing approval, even though it may be explicitly requested
Zeroing the allowance to 0 is a standard situation for the sake of security and situations when it is necessary to take care of the impossibility of withdrawing funds from the contract, but the implementation of interaction with the amount in ArcadeTresuary does not allow this to be done, which is clearly illogical and can lead to unexpected consequences

### PROOF OF CONCEPT
Instances:
* [ArcadeTrasury.sol#L195](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L195)
* [ArcadeTrasury.sol#L217](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L217)
* [ArcadeTrasury.sol#L236](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L236)
* [ArcadeTrasury.sol#L255](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L255)
```js
File: contracts/ArcadeTreasury.sol

189:    function gscApprove(
            address token,
            address spender,
            uint256 amount
        ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {
            if (spender == address(0)) revert T_ZeroAddress("spender");
            if (amount == 0) revert T_ZeroAmount(); // @audit

            // Will underflow if amount is greater than remaining allowance
            gscAllowance[token] -= amount;

            _approve(token, spender, amount, spendThresholds[token].small);
        }


211:    function approveSmallSpend(
            address token,
            address spender,
            uint256 amount
        ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
            if (spender == address(0)) revert T_ZeroAddress("spender");
            if (amount == 0) revert T_ZeroAmount(); // @audit

            _approve(token, spender, amount, spendThresholds[token].small);
        }

230:    function approveMediumSpend(
            address token,
            address spender,
            uint256 amount
        ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
            if (spender == address(0)) revert T_ZeroAddress("spender");
            if (amount == 0) revert T_ZeroAmount(); // @audit

            _approve(token, spender, amount, spendThresholds[token].medium);
        }

249:    function approveLargeSpend(
            address token,
            address spender,
            uint256 amount
        ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
            if (spender == address(0)) revert T_ZeroAddress("spender"); // @audit
            if (amount == 0) revert T_ZeroAmount();

            _approve(token, spender, amount, spendThresholds[token].large);
        }
```

### Mittigation 
Allow interaction with approve in standard practice