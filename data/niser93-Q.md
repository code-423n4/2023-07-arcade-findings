
## Low Risk Issues
|       |Issue  |Instances|
|:-----:|:-----:|:-------:|
|L-01|Loss of precision|3|
|L-02|Setters should have initial value check|4|
|L-03|Return values of ```transfer()```/```transferFrom()``` not checked|2|
|L-04|Execution at deadlines should be allowed|3|

## Not Critical Issues
|       |Issue  |Instances|
|:-----:|:-----:|:-------:|
|NC-01|The ```nonReentrant``` ```modifier``` should occur before all other modifiers|10|
|NC-02|Inconsistent spacing in comments|3|
|NC-03|Consider using ```delete``` rather than assigning zero/false to clear values|13|
|NC-04|Event is not properly ```indexed```|2|
|NC-05|Contract does not follow the Solidity style guide's suggested layout ordering|2|
|NC-06|Missing events in sensitive functions|5|
|NC-07|Zero as a function argument should have a descriptive meaning|3|

# Low Risk Issues
## L-01
### Loss of precision
#### Description
Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator


There are 3 instances of this issue:                

```
File: ARCDVestingVault.sol

  
  330         uint256 unlocked = grant.cliffAmount + (postCliffAmount * blocksElapsedSinceCliff) / totalBlocksPostCliff;

```
[[330](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L330)]

```
File: NFTBoostVault.sol

  
  494         uint128 newVotingPower = (_amount * multiplier) / MULTIPLIER_DENOMINATOR;

  
  633             return (locked * getMultiplier(registration.tokenAddress, registration.tokenId)) / MULTIPLIER_DENOMINATOR;

```
[[494](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L494), [633](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L633)]
## L-02
### Setters should have initial value check
#### Description
Setters should have initial value check to prevent assigning wrong value to the variable. Assignment of wrong value can lead to unexpected behavior of the contract.


There are 4 instances of this issue:                

```
File: ArcadeAirdrop.sol

  @audit: unchecked values _merkleRoot
  75     function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {
  76         rewardsRoot = _merkleRoot;
  77 
  78         emit SetMerkleRoot(_merkleRoot);
  79     }

```
[[75-79](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol#L75-L79)]

```
File: BadgeDescriptor.sol

  @audit: unchecked values newBaseURI
  57     function setBaseURI(string memory newBaseURI) external onlyOwner {
  58         baseURI = newBaseURI;
  59 
  60         emit SetBaseURI(msg.sender, newBaseURI);
  61     }

```
[[57-61](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L57-L61)]

```
File: NFTBoostVault.sol

  @audit: unchecked values tokenAddress, tokenId
  363     function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) public override onlyManager {
  364         if (multiplierValue > MAX_MULTIPLIER) revert NBV_MultiplierLimit();
  365 
  366         NFTBoostVaultStorage.AddressUintUint storage multiplierData = _getMultipliers()[tokenAddress][tokenId];
  367         // set multiplier value
  368         multiplierData.multiplier = multiplierValue;
  369 
  370         emit MultiplierSet(tokenAddress, tokenId, multiplierValue);
  371     }

  @audit: unchecked values newAirdropContract
  392     function setAirdropContract(address newAirdropContract) external override onlyManager {
  393         Storage.set(Storage.addressPtr("airdrop"), newAirdropContract);
  394 
  395         emit AirdropContractUpdated(newAirdropContract);
  396     }

```
[[363-371](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L363-L371), [392-396](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L392-L396)]
## L-03
### Return values of ```transfer()```/```transferFrom()``` not checked
#### Description
This has already reported by [Automatic findings](https://github.com/code-423n4/2023-07-arcade/blob/main/slither/FullReport.md). We report only other instances.
Not all ```IERC20``` implementations ```revert()``` when there's a failure in ```transfer()```/```transferFrom()```. The function signature has a ```boolean``` return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually making a payment


There are 2 instances of this issue:                


```
File: ArcadeTreasury.sol

  
  367             payable(destination).transfer(amount);

```
[[367](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L367)]


```
File: ReputationBadge.sol

  
  171         payable(recipient).transfer(balance);

```
[[171](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L171)]
## L-04
### Execution at deadlines should be allowed
#### Description
This has already reported by [Automatic findings](https://github.com/code-423n4/2023-07-arcade/blob/main/slither/FullReport.md). Anyway, it report only "dangerous comparison".
The condition may be wrong in these cases, as when block.timestamp is equal to the compared > or < variable these blocks will not be executed.


There are 3 instances of this issue:                

```
File: ArcadeToken.sol

  
  146         if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);

```
[[146](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L146)]

```
File: ArcadeTreasury.sol

  
  308         if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {

```
[[308](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L308)]

```
File: ReputationBadge.sol

  
  108         if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));

```
[[108](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L108)]


# Not Critical Issues
## NC-01
### The ```nonReentrant``` ```modifier``` should occur before all other modifiers
#### Description
This is a best-practice to protect against reentrancy in other modifiers


<details>
<summary>There are 10 instances of this issue:</summary>

```
File: ArcadeTreasury.sol

  
  108     function gscSpend(
  109         address token,
  110         uint256 amount,
  111         address destination
  112     ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {
  113         if (destination == address(0)) revert T_ZeroAddress("destination");
  114         if (amount == 0) revert T_ZeroAmount();
  115 
  116         // Will underflow if amount is greater than remaining allowance
  117         gscAllowance[token] -= amount;
  118 
  119         _spend(token, amount, destination, spendThresholds[token].small);
  120     }
  121 

  
  130     function smallSpend(
  131         address token,
  132         uint256 amount,
  133         address destination
  134     ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
  135         if (destination == address(0)) revert T_ZeroAddress("destination");
  136         if (amount == 0) revert T_ZeroAmount();
  137 
  138         _spend(token, amount, destination, spendThresholds[token].small);
  139     }
  140 

  
  149     function mediumSpend(
  150         address token,
  151         uint256 amount,
  152         address destination
  153     ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
  154         if (destination == address(0)) revert T_ZeroAddress("destination");
  155         if (amount == 0) revert T_ZeroAmount();
  156 
  157         _spend(token, amount, destination, spendThresholds[token].medium);
  158     }
  159 

  
  168     function largeSpend(
  169         address token,
  170         uint256 amount,
  171         address destination
  172     ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
  173         if (destination == address(0)) revert T_ZeroAddress("destination");
  174         if (amount == 0) revert T_ZeroAmount();
  175 
  176         _spend(token, amount, destination, spendThresholds[token].large);
  177     }
  178 

  
  189     function gscApprove(
  190         address token,
  191         address spender,
  192         uint256 amount
  193     ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {
  194         if (spender == address(0)) revert T_ZeroAddress("spender");
  195         if (amount == 0) revert T_ZeroAmount();
  196 
  197         // Will underflow if amount is greater than remaining allowance
  198         gscAllowance[token] -= amount;
  199 
  200         _approve(token, spender, amount, spendThresholds[token].small);
  201     }
  202 

  
  211     function approveSmallSpend(
  212         address token,
  213         address spender,
  214         uint256 amount
  215     ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
  216         if (spender == address(0)) revert T_ZeroAddress("spender");
  217         if (amount == 0) revert T_ZeroAmount();
  218 
  219         _approve(token, spender, amount, spendThresholds[token].small);
  220     }
  221 

  
  230     function approveMediumSpend(
  231         address token,
  232         address spender,
  233         uint256 amount
  234     ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
  235         if (spender == address(0)) revert T_ZeroAddress("spender");
  236         if (amount == 0) revert T_ZeroAmount();
  237 
  238         _approve(token, spender, amount, spendThresholds[token].medium);
  239     }
  240 

  
  249     function approveLargeSpend(
  250         address token,
  251         address spender,
  252         uint256 amount
  253     ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
  254         if (spender == address(0)) revert T_ZeroAddress("spender");
  255         if (amount == 0) revert T_ZeroAmount();
  256 
  257         _approve(token, spender, amount, spendThresholds[token].large);
  258     }
  259 

  
  333     function batchCalls(
  334         address[] memory targets,
  335         bytes[] calldata calldatas
  336     ) external onlyRole(ADMIN_ROLE) nonReentrant {
  337         if (targets.length != calldatas.length) revert T_ArrayLengthMismatch();
  338         // execute a package of low level calls
  339         for (uint256 i = 0; i < targets.length; ++i) {
  340             if (spendThresholds[targets[i]].small != 0) revert T_InvalidTarget(targets[i]);
  341             (bool success, ) = targets[i].call(calldatas[i]);
  342             // revert if a single call fails
  343             if (!success) revert T_CallFailed();
  344         }
  345     }
  346 

```
[[108-121](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L108-L121), [130-140](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L130-L140), [149-159](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L149-L159), [168-178](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L168-L178), [189-202](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L189-L202), [211-221](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L211-L221), [230-240](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L230-L240), [249-259](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L249-L259), [333-346](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L333-L346)]

```
File: NFTBoostVault.sol

  
  139     function airdropReceive(
  140         address user,
  141         uint128 amount,
  142         address delegatee
  143     ) external override onlyAirdrop nonReentrant {
  144         if (amount == 0) revert NBV_ZeroAmount();
  145         if (user == address(0)) revert NBV_ZeroAddress("user");
  146 
  147         // load the registration
  148         NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[user];
  149 
  150         // if user is not already registered, register them
  151         // else just update their registration
  152         if (registration.delegatee == address(0)) {
  153             _registerAndDelegate(user, amount, 0, address(0), delegatee);
  154         } else {
  155             // if user supplies new delegatee address revert
  156             if (delegatee != registration.delegatee) revert NBV_WrongDelegatee(delegatee, registration.delegatee);
  157 
  158             // get this contract's balance
  159             Storage.Uint256 storage balance = _balance();
  160             // update contract balance
  161             balance.data += amount;
  162 
  163             // update registration amount
  164             registration.amount += amount;
  165 
  166             // sync current delegatee's voting power
  167             _syncVotingPower(user, registration);
  168         }
  169 
  170         // transfer user ERC20 amount only into this contract
  171         _lockTokens(msg.sender, uint256(amount), address(0), 0, 0);
  172     }
  173 

```
[[139-173](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L139-L173)]

            
</details>

## NC-02
### Inconsistent spacing in comments
#### Description
Some lines use ```// x``` and some use ```//x```. The instances below point out the usages that don't follow the majority, within each file. Therefore, with respect [NatFormat](https://docs.soliditylang.org/en/v0.8.20/natspec-format.html), we suggest to use only one space or same indentation for NatSpec comments.


There are 3 instances of this issue:                

```
File: ARCDVestingVault.sol

  
  297     /**
  298      * @notice Getter function for the grants mapping.
  299      *
  300      * @param who            The owner of the grant to query
  301      *
  302      * @return               The user's grant object.
  303      */

```
[[297-303](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L297-L303)]

```
File: NFTBoostVault.sol

  
  409     /**
  410      * @notice A function to access the storage of the nft's voting power multiplier.
  411      *
  412      * @param tokenAddress              The address of the ERC1155 token to set the
  413      *                                  multiplier for.
  414      * @param tokenId                   The token id of the ERC1155 for which the multiplier is being set.
  415      *
  416      * @return                          The token multiplier.
  417      */

  
  618     /**
  619      * @dev Helper that returns the current voting power of a registration.
  620      *
  621      * @dev This is not always the recorded voting power since it uses the latest multiplier.
  622      *
  623      * @param registration               The registration to check for voting power.
  624      *
  625      * @return                           The current voting power of the registration.
  626      */

```
[[409-417](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L409-L417), [618-626](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L618-L626)]
## NC-03
### Consider using ```delete``` rather than assigning zero/false to clear values
#### Description
The ```delete``` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic


<details>
<summary>There are 13 instances of this issue:</summary>

```
File: ARCDVestingVault.sol

  
  133         grant.withdrawn = 0;

  
  178         grant.allocation = 0;

  
  179         grant.cliffAmount = 0;

  
  180         grant.withdrawn = 0;

  
  181         grant.created = 0;

  
  182         grant.expiration = 0;

  
  183         grant.cliff = 0;

  
  184         grant.latestVotingPower = 0;

```
[[133](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L133), [178](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L178), [179](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L179), [180](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L180), [181](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L181), [182](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L182), [183](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L183), [184](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L184)]

```
File: NFTBoostVault.sol

  
  251             registration.amount = 0;

  
  252             registration.latestVotingPower = 0;

  
  253             registration.withdrawn = 0;

  
  499         registration.withdrawn = 0;

  
  566         registration.tokenId = 0;

```
[[251](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L251), [252](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L252), [253](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L253), [499](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L499), [566](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L566)]

            
</details>

## NC-04
### Event is not properly ```indexed```
#### Description
Index event fields make the field more quickly accessible [to off-chain tools](https://ethereum.stackexchange.com/questions/40396/can-somebody-please-explain-the-concept-of-event-indexing) that parse events. This is especially useful when it comes to filtering based on an address. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Where applicable, each ```event``` should use three ```indexed``` fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three applicable fields, all of the applicable fields should be indexed.


There are 2 instances of this issue:                

```
File: ArcadeToken.sol

  
  102     event MinterUpdated(address newMinter);

```
[[102](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L102)]

```
File: ArcadeTokenDistributor.sol

  
  64     event Distribute(address token, address recipient, uint256 amount);

```
[[64](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L64)]
## NC-05
### Contract does not follow the Solidity style guide's suggested layout ordering
#### Description
This has already reported by [Automatic findings](https://gist.github.com/itsmetechjay/e494eb18a34459c4d7841fc6fdc700e1). We report only other instances.
The [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) says that, within a contract, the ordering should be 1) Type declarations, 2) State variables, 3) Events, 4) Modifiers, and 5) Functions, but the contract(s) below do not follow this ordering


There are 2 instances of this issue:                


```
File: BaseVotingVault.sol

  
  187     modifier onlyManager() {
  188         if (msg.sender != manager()) revert BVV_NotManager();
  189 
  190         _;
  191     }

```
[[187-191](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L187-L191)]

```
File: NFTBoostVault.sol

  
  701     modifier onlyAirdrop() {
  702         if (msg.sender != Storage.addressPtr("airdrop").data) revert NBV_NotAirdrop();
  703 
  704         _;
  705     }

```
[[701-705](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L701-L705)]
## NC-06
### Missing events in sensitive functions
#### Description
Events should be emitted when sensitive changes are made to the contracts, but some functions lack them.


There are 5 instances of this issue:                

```
File: ArcadeTokenDistributor.sol

  
  174     function setToken(IArcadeToken _arcadeToken) external onlyOwner {
  175         if (address(_arcadeToken) == address(0)) revert AT_ZeroAddress("arcadeToken");
  176         if (address(arcadeToken) != address(0)) revert AT_TokenAlreadySet();
  177 
  178         arcadeToken = _arcadeToken;
  179     }

```
[[174-179](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L174-L179)]

```
File: BaseVotingVault.sol

  
  68     function setTimelock(address timelock_) external onlyTimelock {
  69         if (timelock_ == address(0)) revert BVV_ZeroAddress("timelock");
  70 
  71         Storage.set(Storage.addressPtr("timelock"), timelock_);
  72     }

  
  80     function setManager(address manager_) external onlyTimelock {
  81         if (manager_ == address(0)) revert BVV_ZeroAddress("manager");
  82 
  83         Storage.set(Storage.addressPtr("manager"), manager_);
  84     }

```
[[68-72](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L68-L72), [80-84](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L80-L84)]

```
File: NFTBoostVault.sol

  
  305     function updateNft(uint128 newTokenId, address newTokenAddress) external override nonReentrant {
  306         if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);
  307 
  308         if (IERC1155(newTokenAddress).balanceOf(msg.sender, newTokenId) == 0) revert NBV_DoesNotOwn();
  309 
  310         NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];
  311 
  312         // If the registration does not have a delegatee, revert because the Registration
  313         // is not initialized
  314         if (registration.delegatee == address(0)) revert NBV_NoRegistration();
  315 
  316         // if the user already has an ERC1155 registered, withdraw it
  317         if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
  318             // withdraw the current ERC1155 from the registration
  319             _withdrawNft();
  320         }
  321 
  322         // set the new ERC1155 values in the registration and lock the new ERC1155
  323         registration.tokenAddress = newTokenAddress;
  324         registration.tokenId = newTokenId;
  325 
  326         _lockNft(msg.sender, newTokenAddress, newTokenId, 1);
  327 
  328         // update the delegatee's voting power based on new ERC1155 nft's multiplier
  329         _syncVotingPower(msg.sender, registration);
  330     }

  
  342     function updateVotingPower(address[] calldata userAddresses) public override {
  343         if (userAddresses.length > 50) revert NBV_ArrayTooManyElements();
  344 
  345         for (uint256 i = 0; i < userAddresses.length; ++i) {
  346             NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[userAddresses[i]];
  347             _syncVotingPower(userAddresses[i], registration);
  348         }
  349     }

```
[[305-330](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L305-L330), [342-349](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L342-L349)]
## NC-07
### Zero as a function argument should have a descriptive meaning
#### Description
Consider using descriptive constants or an enum instead of passing zero directly on function calls, as that might be error-prone, to fully describe the caller's intention.


There are 3 instances of this issue:                

```
File: NFTBoostVault.sol

  
  153             _registerAndDelegate(user, amount, 0, address(0), delegatee);

  
  171         _lockTokens(msg.sender, uint256(amount), address(0), 0, 0);

  
  286         _lockTokens(msg.sender, amount, address(0), 0, 0);

```
[[153](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L153), [171](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L171), [286](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L286)]

