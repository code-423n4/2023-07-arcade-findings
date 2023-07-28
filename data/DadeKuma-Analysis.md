# Arcade.xyz Analysis

## Summary

|Id|Title|
|:--:|:-------|
|[01](#01-high-level-architecture)| High-level architecture |
|[02](#02-analysis-of-the-codebase)| Analysis of the codebase |
|[03](#03-architecture-feedback)| Architecture feedback |
|[04](#04-centralization-risks)| Centralization risks |
|[05](#05-systemic-risks)| Systemic risks |

## [01] High-level architecture

![Arcade.xyz architecture](https://i.imgur.com/GBHt7g6.png)

## [02] Analysis of the codebase

- The overall architecture of Arcade.xyz demonstrates a well-structured and modular design, aligning with widely accepted governance patterns.
- The integration of the Council Framework provides flexibility, though some parts of the code could benefit from refactoring, as they were developed a few years ago and the best practices has changed.
- Codebase quality is **high**, with extensive comments that thoroughly explain each step in the process.
- Test quality is **high**, featuring well-structured tests with extensive coverage (99%). However, incorporating fuzzing tests would further enhance test robustness.

## [03] Architecture feedback

- A crucial aspect that demands attention in the Arcade.xyz architecture is the absence of a Time-Weighted Average Price (TWAP) mechanism during governance votes.
- Without a TWAP, the voting power of token holders may be vulnerable to sudden price fluctuations, potentially leading to vote manipulation by malicious users.
- The use of storage pointers represents a clever design choice that reduces gas consumption, ultimately benefiting users.
- The choice of using ERC-1155 as a multiplier for voting power is interesting, as utilizing tokens in this manner introduces an innovative approach to enhance the governance process. 

## [04] Centralization risks

Arcade.xyz faces several centralization risks that necessitate mitigation to ensure a fair and decentralized environment for all users.
- The existence of privileged roles and access in the Vault contracts and Core Voting Contracts introduces potential centralization risks. For instance, the manager role in Vaults grants access to critical operational functions, while the timelock role can alter managers and its own role.
- To prevent centralization of power, it is crucial to distribute these privileged roles fairly and responsibly among various stakeholders.
- The disproportionate power of the owner in certain contracts, such as `ArcadeGSCVault`, requires balancing to ensure equitable decision-making in governance.
- Currently, privileged addresses have the ability to wield more power than regular users when participating in proposal voting. This discrepancy in voting power raises concerns about fairness and decentralization within the governance process.

## [05] Systemic risks

- The `ArcadeTreasury` contract's ability to execute arbitrary calls without limits necessitates the implementation of strict limitations and safeguards, such as a whitelist approach to authorize only approved contracts.
- Moreover, the reliance on several external components raises concerns regarding unaudited code. Ensuring the security of these components is critical, considering their significant impact on Arcade.xyz's core functionality.
- `NFTBoostVault` presents a potential vulnerability as it allows any ERC-1155 token to be deposited. The interaction between the vault and tokens must be carefully secured to prevent any unintended consequences or malicious behaviors.

### Time spent:
22 hours