# Analysis - Arcade.xyz 
# Summary

| List |Head |Details|
|:--|:----------------|:------|
|1 | Overview of Arcade.xyz platform| overview of the key components and features of Arcade.xyz  |
|2 | My Thoughts | My own thoughts about future of the this protocol |
|3 |Audit approach | Process and steps i followed  |
|4 |Learnings | Learnings from this protocol|
|5 |Possible Systemic Risks | The possible systemic risks based on my analysis |
|6 |Code Commentary | Suggestions for existing code base |
|7 |Centralization risks | Concerns associated with centralized systems |
|8 |Gas Optimizations | Details about my gas optimizations findings and gas savings  |
|9 |Possible Risks as per my analysis | Possible risks |
|10 |Time spent on analysis  | The Over all time spend for this reports |

## Overview
Arcade.xyz is a platform for autonomous borrowing, lending, and escrow of NFT collateral on EVM blockchains. This governance system is built on the ``Council Framework``

Arcade governance classified in 4 types 

  - ``Voting Vaults`` :
     Each voting vault contract is a separate deployment, which handles its own deposits and vote-counting mechanisms for those deposits
  - ``Core Voting``:
     These contracts can be used to submit and vote on proposed governance transactions.core voting contracts may either administrate the protocol directly, or may be intermediated by a Timelock contract
  - ``Token``:
     The ERC20 governance token, along with contracts required for initial deployment and distribution of the token (airdrop contract, initial distributor contract)   
  - ``NFT``: The ERC1155 token contract along with its tokenURI descriptor contract. The ERC1155 token used in governance to give a multiplier to a user's voting power  
     

### My Thoughts
Arcade.xyz is a platform change the future in following areas

 1. it provides a way for users to have a say in how the protocol is governed. This is important because it gives users more control over their assets and ensures that the protocol is aligned with their interests

 2. The Arcade governance system is designed to be scalable. This means that it can be used to govern large and complex DeFi protocols. This is important because it will allow DeFi protocols to grow and evolve without sacrificing decentralization.

 3. The Arcade governance system is secure. This is because it is built on the Council Framework, which is a battle-tested governance system. This means that the Arcade governance system is less likely to be exploited by malicious actors


## Audit approach

I followed below steps while analyzing and auditing the code base.

1. Read the contest Readme.md and took the required notes.

  - Arcade.xyz platform uses 
   - Inheritance
   - NFT
   - Timelock Functions
   - ERC20
   - The test coverage is 99%

2. Analyzed the over all codebase one iterations very fast

3. Study of documentation to understand each contract purpose, its functionality, how it is connected with other contracts, etc.

4. Then i read old audits and already known findings. Then go through the bot races findings 

5. Then setup my testing environment things. Run the tests to checks all test passed.

6. Finally, I started with the auditing the code base in depth way I started understanding line by line code and took the necessary notes to ask some questions to sponsors.

## Stages of audit

- ``The first stage of the audit``

During the initial stage of the audit for Arcade.xyz platform, the primary focus was on analyzing gas usage and quality assurance (QA) aspects. This phase of the audit aimed to ensure the efficiency of gas consumption and verify the robustness of the platform.

- ``The second stage of the audit``

In the second stage of the audit for Arcade.xyz platform, the focus shifted towards understanding the protocol usage in more detail. This involved identifying and analyzing the important contracts and functions within the system. By examining these key components, the audit aimed to gain a comprehensive understanding of the protocol's functionality and potential risks. 

- ``The third stage of the audit``

During the third stage of the audit for Arcade.xyz platform, the focus was on thoroughly examining and marking any doubtful or vulnerable areas within the protocol. This stage involved conducting comprehensive vulnerability assessments and identifying potential weaknesses in the system. Found ``60-70`` ``vulnerable`` and ``weakness`` code parts all marked with ``@audit tags``.

- ``The fourth stage of the audit``

During the fourth stage of the audit for Arcade.xyz platform, a comprehensive analysis and testing of the previously identified doubtful and vulnerable areas were conducted. This stage involved diving deeper into these areas, performing in-depth examinations, and subjecting them to rigorous testing, including fuzzing with various inputs. Finally concluded findings after all research's and tests. Then i reported C4 with proper formats. 


## Learnings

 Arcade.xyz's governance system, we can understand several key learnings

1. ``Decentralized Governance`` : Arcade.xyz allows token holders to vote on platform decisions, ensuring that no single entity or group has complete control. It's a democratic way of making choices for the platform.

2. ``Voting Vaults and Delegation``: Users can deposit their voting tokens in separate vaults and also delegate their voting power to someone they trust, making it easier for more people to participate in voting.

3. ``ERC20 and ERC1155 Tokens``: Arcade.xyz uses two types of tokens - ERC20 for basic voting and ERC1155 for more advanced governance features, like giving some tokens more voting influence.

4. ``Council Framework`` : The governance system follows a specific Council Framework, which outlines rules and guidelines for how decisions are made.

5. ``Protocol Administrations and Timelock`` : Proposals can directly impact the platform or go through a Timelock first, adding a delay to allow community review and transparency.

6. ``Autonomy and Smart Contracts``: Once set up, the governance system operates autonomously using smart contracts, without needing central control.

7. ``Transparency and Openness``: Arcade.xyz is transparent about how things work, providing clear technical details for users to understand and participate effectively.

8. ``NFT Integration`` : Non-fungible tokens (NFTs) can enhance voting power, incentivizing community involvement and participation.


## Possible Systemic Risks

Centralization Risk: Voting power could concentrate among a few, leading to centralization.

Delegate Control Concerns: Delegation may reduce voter engagement and representativeness.

Low Participation: Low turnout may undermine governance legitimacy.

Ineffective Proposals: Poorly designed proposals may hinder progress.

Voting Manipulation: Malicious actors may try to influence outcomes.

Token Concentration: Unequal token distribution may skew decisions.

NFT Impact Uncertainty: ERC1155 NFTs' effects may have unintended consequences.

Technical Vulnerabilities: Smart contract bugs could compromise governance.

Community Disagreements: Conflicts may impede decision-making.

Governance Rule Updates: Changing rules may be challenging and require consensus.


## Code Commentary

1. Instead of using bytes32 constants for role identifiers, consider using an enum for better readability and code organization
2. Functions like smallSpend, mediumSpend, and largeSpend have similar logic. Consider refactoring the code to combine the common logic into a single internal function and call it from each respective function. This can reduce code duplication
3. For events like TreasuryTransfer and TreasuryApproval, consider emitting only relevant data instead of emitting the entire amount and destination/spender
4. immutable variable should be in upper case 
5. Instead of using error messages in revert statements, consider using an enum to define error codes. This can make error handling more structured and easier to understand
6. Ensure that error messages are clear and informative to help developers and users understand contract behavior and potential issues
7. The contract could benefit from more detailed comments and documentation to explain the purpose of various functions, their expected behavior, and any potential risks or constraints. This would make it easier for developers to understand and interact with the contract
8. Consider reordering the modifiers in the function declarations to improve readability. For example, move the onlyManager modifier to be the last one, as it's the most specific and should be applied after other access control checks.
9. Consider using a struct to represent grant data instead of individual storage variables. This can make the code cleaner and easier to manage
10.  Instead of importing the entire ERC20 contract, consider using an ERC20 interface with only the necessary functions. This helps to reduce contract deployment costs.
11. The contract uses a mix of camelCase and snake_case for function and variable names. It's advisable to use a consistent naming convention throughout the contract to enhance readability.
12. There are some duplicated code segments (e.g., handling delegation and voting power updates) that could be abstracted into separate helper functions to improve code readability and maintainability.
13. While the contract has some comments, additional documentation could be provided to clarify the purpose and behavior of certain functions and variables.
14.In functions where multiple events are emitted, consider consolidating them into a single emit statement for better gas efficiency

## Centralization risks

A single point of failure is not acceptable for this project Centrality risk is high in the project, the role of ``onlyOwner``detailed below has very critical and important powers

Project and funds may be compromised by a malicious or stolen private key ``onlyOwner`` ``msg.sender``

```solidity
File: contracts/nft/BadgeDescriptor.sol

57:     function setBaseURI(string memory newBaseURI) external onlyOwner {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L57

```solidity
File: contracts/token/ArcadeAirdrop.sol

62:     function reclaim(address destination) external onlyOwner {

75:     function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol#L62

```solidity

File: contracts/token/ArcadeTokenDistributor.sol

73:     function toTreasury(address _treasury) external onlyOwner {

89:     function toDevPartner(address _devPartner) external onlyOwner {

105:     function toCommunityRewards(address _communityRewards) external onlyOwner {

121:     function toCommunityAirdrop(address _communityAirdrop) external onlyOwner {

138:     function toTeamVesting(address _vestingTeam) external onlyOwner {

155:     function toPartnerVesting(address _vestingPartner) external onlyOwner {

174:     function setToken(IArcadeToken _arcadeToken) external onlyOwner {

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L73

## Gas Optimizations

During the audit of the smart contracts on ``Arcade.xyz's`` governance platform, several key gas optimization issues were identified. These issues could potentially lead to increased transaction costs and inefficiencies on the Ethereum network. One of the notable findings was related to state variables, where it was observed that they can be packed together to ``utilize fewer storage slots``. By organizing related variables in this way, the number of storage operations, such as ``SSTORE`` and ``SLOAD`` opcodes, can be reduced, resulting in significant gas savings.

Another prevalent optimization opportunity involved caching state variables in stack variables instead of repeatedly re-reading them from storage. This approach can eliminate redundant storage operations, leading to improved gas efficiency. Utilizing opcodes like ``SWAP``, ``DUP``, and ``MLOAD`` efficiently allows for cost-effective access to state variables during contract execution.

In addition, a gas-saving technique was found in arithmetic operations on state variables. It was observed that using the = operator rather than += and -= could result in reduced gas consumption for the ADD, SUB, and MSTORE opcodes. Moving IF statements or require() functions that check input arguments to the top of the functions was another recommendation to avoid unnecessary code execution, thereby saving gas. This optimization takes advantage of the`` JUMP`` and ``JUMPI`` opcodes, which control the flow of execution within the contract.

Furthermore, multiple accesses of mappings or arrays were identified as potential areas for improvement. By employing local variable caches, redundant read operations from storage can be minimized, reducing gas costs. The relevant opcodes in this context include ``MLOAD``, ``MSTORE``, and ``SLOAD``.

Moreover, it was noticed that emitting state variables when stack variables are available could lead to unnecessary gas consumption. Emitting state variables involves storage operations, while stack variables are more gas-efficient. The LOG opcode is used for emitting event logs.

Lastly, optimizing function parameters by using calldata instead of memory can save gas. The ``CALLDATASIZE`` opcode is used to access the size of the input data, and ``CALLDATALOAD`` is used to retrieve specific function parameters from the input.

https://code4rena.com/contests/2023-07-arcadexyz/submit?issue=151

## Possible Risks as per my analysis

- The ``batchCalls`` function allows executing multiple low-level calls in a single transaction. If a large number of calls are included or if any of the calls fail, the entire transaction will be reverted, leading to gas wastage. This can result in higher transaction costs and failed transactions due to gas limits.

- The ``smallSpend``, ``mediumSpend``, ``largeSpend``, ``approveSmallSpend``, ``approveMediumSpend``, and ``approveLargeSpend`` functions do not perform input validation to check whether the ``amount`` parameter exceeds the token balance or allowance. This could lead to unintended spending or approval of more tokens than the treasury holds or intends to allow.

- The ``_spend`` and `` _approve`` functions enforce a per-block spending/approval limit (limit) to avoid excessive spending. However, there is no mechanism to reset the ``blockExpenditure`` storage variable when a new block starts, which could lead to inconsistencies if the contract is active across multiple blocks.

- In the ``_spend`` function, if the destination is a contract without a ``receive function``, the transfer will fail, resulting in a ``loss of funds``. The function should include checks to handle such cases appropriately.

- Hardcoded ``cooldown period`` may cause problem in future

- Lack of same value input control in critical ``setMinter()`` functions

- Use safeInceaseAllowance()/safeDecreaseAllowance() instead of approve() or deprecated safeApprove() Functions

- Don't use payable(address).tranfer() to avoid unexpected fund lose

- The function addGrantAndDelegate allows the manager to create a new grant without checking if the grant recipient already has an active grant. If a user already has an active grant, this function will overwrite the existing grant, which might lead to unintended behavior. It would be better to check if the recipient already has a grant and handle this situation accordingly.


- In the _getWithdrawableAmount function, there are multiple arithmetic calculations involving subtraction, addition, and multiplication. Ensure that these calculations do not result in underflow or overflow situations, which could lead to unintended behavior or vulnerabilities

- The contract uses a custom state management system with the Storage library, which might make it harder to understand for developers not familiar with this approach. Consider using standardized storage patterns to improve code readability and maintainability.

- Some functions, such as _lockNft, _lockTokens, and _grantVotingPower, are marked as internal but are not strictly necessary to be internal. It's essential to assess whether these functions need to be called from other contracts or external interactions

 ## Time spent on analysis 
``15 Hours``

















































### Time spent:
15 hours