# Low Risk Findings

## [L‑01] Use Ownable2Step rather than Ownable
`Ownable2Step` and `Ownable2StepUpgradeable` prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.

23:`contract ArcadeTokenDistributor is Ownable {`
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L23


# Non-Critical Finding
## [N-01] For modern and more readable code; update import usages

>Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.
> ### *Recommendation*
> `import {contract1 , contract2} from "filename.sol";`

5: `import "./external/council/CoreVoting.sol";`
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeGSCCoreVoting.sol#L5

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L5C1-L7C16
## [N-02] Use a non-legacy and more battle-tested version
Use 0.8.10

## [S-03] Generate perfect code headers every time
Description:
I recommend using header for Solidity code layout and readability

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## [N-04] NatSpec comments should be increased in contracts
> all contest

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity Official documentation

in complext project such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## [N-05] Include return parameters in NatSpec comments
> all contest

