# Analysis  - Arcade.xyz Project 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Test analysis | Test scope of the project and quality of tests |
|c) |Architectural | Architecture feedback |
|d) |Documents  | What is the scope and quality of documentation for Users and Administrators? |
|e) |Centralization risks | How was the risk of centralization handled in the project, what could be alternatives? |
|f) |Systemic risks | Potential systemic risks in the project |
|g) |Competition analysis| What are similar projects? |
|h) |Security Approach of the Project | Audit approach of the Project |
|i) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|j) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|k) |New insights and learning from this audit | Things learned from the project |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-07-arcade#scope

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-07-arcade#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Arcade Protocol](https://docs.arcade.xyz/docs) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither](https://github.com/code-423n4/2023-07-arcade/blob/main/slither/FullReport.md)| |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-07-arcade#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-07-arcade#scope)|Top-down analysis of codes according to architectural design, IDE used: VsCode|
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-07-arcade#additional-context-privileged-roles--access)||


## b) Test analysis
Test coverage is 99%

What could they have done better?;
There are many unit tests in the project, integration tests in which the interaction of contracts with each other are modeled should be increased.
The test scope is written in Typescript and the Hardhat framework is used, I recommend that the tests are also written in Solidity with the Foundry framework,
Foundry is the smart contract development kit. It is used for the fast development of contracts, easy testing of smart contracts, and easy deployment of smart contracts.

Advantages of Foundry;
Writing tests in the first layer directly with Solidity models reality better
fast to develop
Built-in Fuzzing
Solidity based testing
EVM cheat codes
Scripts based in bash/shell

## c) Architectural 

#### Asset Vaults
Asset Vault - Create a Vault and deposit one or multiple NFT(s) into the Vault. Vault owners receive an ERC721 token representing ownership of the Vault. Vaults can hold any or all of ETH, ERC20s, ERC721, ERC1155, and/or CryptoPunks.
Airdrop Utility - Receive airdrops for NFT assets held in a Vault.
Delegate Cash - Enables Vault holders to assign a hot wallet delegate for NFT assets deposited into a Vault on Arcade.xyz.
Staking - Bored Ape Yacht Club (BAYC), Mutant Ape Yacht Club (MAYC) and APECoin ($APE) holders can mint and deposit assets into a Staking Vault and then stake into a supported pool (per Horizen Labs) and obtain liquidity on these assets through the Arcade.xyz platform.

#### Borrowing
Set Terms - Asset owners can set terms on their Vault or NFT - specifying (ERC20), duration, interest rate, for term loans of positive duration, collateralized by the assets in the Vault or NFT.
Trustless Matching - Lenders can fund a loan request trustlessly and once countersigned, either lender or borrower can marshal a loan for on-chain settlement.
Borrower Note - Upon settling a loan on-chain, the funding amount is disbursed to the borrower's address, the encapsulated assets in a Vault are held in escrow by the protocol, and the borrower receives a borrower note (ERC721) that represent their claim to the assets upon loan repayment.
No Prepayment Penalty - Loans can be paid off any time to recoup assets held in escrow on-chain with the protocol.

#### Lending
Offers - Lenders can make offers on Vaults or NFTs.
Collection Offers - Lenders can also make offers on many of the popular collections in the space.
Trustless Matching - Borrowers can settle a loan by accepting a loan offer trustlessly since the Lender has digitally signed the terms of the offer and approved the funding amount for those terms to the Arcade Protocol.
Lender Note - Upon settling a loan on-chain, the lender transfers the funding amount from their wallet to the borrower's and receive a lender note (ERC721) that represents their claim on the Vault encapsulating the assets collateralizing the loan (and therefore a claim on the assets themselves) if the loan were to default.
Trustless Claims - Simple logic embedded in the Arcade Protocol dictates whether a loan is in default. If the principal and interest amount have not been paid by the end of the loan duration, the loan is claimable. The Vault, and therefore the underlying assets collateralizing the defaulted loan, are transferred on-chain to the address that owns the lender note.



## d) Documents 
Documentation should be increased further, it is recommended to add the architectural design to the documents as infographic


##  e) Centralization risks 
There is a risk of centrality in the project as the onlyOwner
The owner role has a single point of failure and onlyOwner can use critical functions, posing a centralization issue. There is always a chance for owner keys to be stolen, and in such a case, the attacker can cause serious damage to the project due to important functions.

The code detail of this topic is specified in automatic finding;
https://gist.github.com/itsmetechjay/e494eb18a34459c4d7841fc6fdc700e1#m-6-the-owner-is-a-single-point-of-failure-and-a-centralization-risk

##  f) Systemic risks 
The biggest systemic risk of the project; It is the central data risk inherent in NFTs. As it is known, in a significant part of NFTs, metadata is stored in centralized networks, even in networks such as decentralized IPFS, there is a risk that these metadata will be lost if you do not use a pinning service, in short, every non-onchain metadata is risky and it has a systemic risk because the main field of activity in the project is NFTs.

## g) Competition analysis
Similar protocols ; 
https://nftfi.com/
https://collateralnetwork.io/



## h) Security Approach of the Project

Successful current security understanding of the project;
1- First, an audit was made from an audit firm (Omniscia)
1- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.



What the project should add in the understanding of Security;
1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)
2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)
3- After the inspection, the concept of "continuous inspection" should be adhered to with bug bounty programs such as ImmuneFi.


## ji) Other Audit Reports and Automated Findings 

**Other Audit Reports (Omniscia):**
https://github.com/code-423n4/2023-07-arcade/tree/main/audits

**Slither Report:**
https://github.com/code-423n4/2023-07-arcade/blob/main/slither/FullReport.md

**Automated Findings:**
https://gist.github.com/thebrittfactor/0cfcacbd1366b927c268d0c41b86781b

Especially Medium-High detections in the Automated Finding Report should be taken into account;

## Medium Risk Issues

| Number | Issues                                                                      | Instances |
| :----: | :-------------------------------------------------------------------------- | :-------: |
| [M-01] | Approve zero first                                                          |     1     |
| [M-02] | The surplus ether is not returned                                           |     1     |
| [M-03] | Use `_safeMint` instead of `_mint` for ERC721                               |     1     |
| [M-04] | Use `.call()` instead of `.send()` or `.transfer()`                         |     2     |
| [M-05] | Contracts are vulnerable to fee-on-transfer token related accounting issues |    17     |
| [M-06] | The owner is a single point of failure and a centralization risk            |    24     |
| [M-07] | Unsafe ERC20 operations                                                     |     2     |
| [M-08] | ERC20 return values not checked                                             |     2     |


## k) Gas Optimization
The project is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, but gas optimizations will not be a priority considering code readability and code base size




## l) New insights and learning from this audit 

- The importance of having a clear and well-defined governance structure. The Arcade governance system has a clear and well-defined structure, which helps to ensure that the protocol is governed in a fair and transparent way.
- The need for a variety of voting mechanisms. The Arcade governance system uses a variety of voting mechanisms, which helps to ensure that all members of the community have a voice in the governance process.
- The importance of security and transparency. The Arcade governance system is designed to be secure and transparent, which helps to protect the protocol from attack and ensures that everyone can see how the system works.
- I learned information and technical details about how such an NFT lending and borrowing project should have API integration according to develeopers.
https://docs.arcade.xyz/docs/api-introduction-and-getting-started

### Time spent:
11 hours