## 1 Any comments for the judge to contextualize your findings
Findings largely focuses on ArcadeToken and ArcadeAirdrop

## 2. Approach taken in evaluating the codebase

A total of 5 days were spent to cover this audit, broken down into the following:

1st Day: Understand protocol docs, action creation flow and policy management
2nd Day: Focus on linking docs logic to Voting Vaults and Token Contracts coupled with typing reports for vulnerabilities found
3rd Day: Focus on different types of strategies contract coupled with typing reports for vulnerabilities found
4th Day: Sum up audit by completing QA analysis and high med

## 3. Architecture Improvements

### 1): Storage pointers
Splitting into separate contracts for the contracts that are not upgradeable, as opposed to the storage provided by the council, and using pointers and assemblies to access them is unnecessary and overkill; it could have been accomplished by using the storage variables as council are upgradable and arcade is not, upgradable pattern should be avoided to make code cleaner

### 2): Standary library for reentrancy
The code isn't using the standary library of openzeppelin for re-entrancy instead a storage assembly based is used which shouldn't be as openzeppelin is battle tested better use consistently that one,  instead of using this use the openzeppelin one
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/libraries/HashedStorageReentrancyBlock.sol

### 3): Multi-Sig
There are alot of roles in the contract but one of them is minter role which should be multi-sig as lots of roles are multi-sig so it is better to change it to multi-sig also to avoid centralization issue

## 4. Codebase quality analysis
### Codebase is mostly well organized but there are some disparencies that can be removed:

Arcade uses alot of the council code and follows the same pattern. One of the pattern that is being followed is using 
storage pointers to store every thing and than there is history pattern that keeps array of a spevific variable at specific location
and new values are pushed into array instead of changing the existing value.

The code for which can be seen in following few files:
[History.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/external/council/libraries/History.sol)
[Storage.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/external/council/libraries/Storage.sol)
[NFTBoostVaultStorage.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/libraries/NFTBoostVaultStorage.sol)
[ARCDVestingVaultStorage.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/libraries/ARCDVestingVaultStorage.sol)

Lets discuss the history.sol. It keeps record of all the historic values for a certain variable. For example in code we track the 
delegate voting power with history. And than load the latest and make changes to it and than push the latest to top of array. That is
a lot of functionality and byte code for relatively very simple ta task. Protocol could have just simply used a mapping for it instead
of simply going with the architecture of council.

Same goes for all the storage pattern, as discussed prior, arcade is non upgradeable and apart from that though it works still it would make
code a lot better, cleaner and readable if simple storage variables would have been used.

## 5. Centralization risks

Centralization risks are mostly minimized due to use of the multisig and governance for the important roles, but there are some roles than can lead to centralization, like minter role in arcade token.

## 7. Systemic risks
One of the possible systemic risks that the protocol may face is the integration with council. Even though the code of council has gone through rigorous testing, there is always a risk involved when integrating with a foreign code. This risk is unavoidable as such parts are not typically included in security audits. However, it is important to note that the risks associated with such integration can be mitigated through extensive testing and security audits. Thus, it is crucial to ensure that all necessary precautions are taken before integrating with council. This will help to minimize the potential risks and ensure the overall security and stability of the protocol.


### Time spent:
30 hours