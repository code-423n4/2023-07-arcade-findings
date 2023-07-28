**Any comments for the judge to contextualize your findings**
First time diving into the Council framework, which is a very interesting take on motivating governance participation.

Still guessing what is expected for this analysis, hopefully it's near the mark.



**Approach taken in evaluating the codebase**
Familiarisation with the Council framework through their [documentation](https://docs.element.fi/governance-council/council-protocol-overview/voting-vaults),
then a mental mapping to place the parts of Aracade.xyz that extended the framework, aided by the C4 breakdown and scope descriptions.

With the high level understanding, then I proceeded with a line by line analysis, marking with inline comments area of 
interest and potential issues. After a complete pass a write up of the QA and this Analysis, then onto experimentation
on the earlier marked areas for investigation for potential higher importance issues.



**Architecture recommendations**
With the Council Framework being quite opinionated, any implementation is limited in deviation from their prescripted
architecture. That being said, the design taken by Arcade.xyz is clean with the contracts appropriately encapsulated 
and leveraging the Council correctly.



**Codebase quality analysis**
The codebase is generally at a very high standard.

There's plenty of decent quality NatSpec comments, and when present they meaningfully describes the purpose
or ranges of the function, parameter or return value rather than merely re-stating the name in a sentence
adding no additional depth of meaning.

Code style is consistently across the contracts (with only a few minor exceptions), including sensible function, parameter 
and local variables names. Combined with concise function size (through splitting code up with internal or private 
functions) providing an easier to understand code base.

The test coverage is also worth commending, note only for their volume, but also the variety of use cases covered. 

It is also great to see the ASCII art in `ArcadeToken`, although a nit for that would be Arcade has a lowercase `a` 
rather than the uppercase `A` seen elsewhere.



**Centralization risks**
The difficulty with the contracts under audit, are they rely on a wider system, where the centralisation risks primarily
steam from or be manage by e.g. Timelock used for GSC operations.

Primarily the risks are from access control, precisely how the access control address is managed and what it is being 
beyond the scope of the audit.
An EOA will present different risks than a multisig, which are different again to a timelock and there are configuration
within each that affect the conversation further.

`ArcadeToken.minter` has the ability to mint tokens, but is limited by a time delay and the 2% inflation cap on existing supply.

`ArcadeTokenDistributor.owner` can choose which addresses get the initial 100m arcade tokens, set the token contract and 
as OZ Ownable is used, initially the deploying address will be the owner.

`ARCDVestingVault.manager` has control over grant creation, grant revoking and withdrawing of any unassigned tokens.

`NFTBoostVault.manager` can change the Airdrop contract address, but more importantly set the nft multiplier 
i.e. which NFT address to tokenId combiation receive which multiplier.


`NFTBoostVault` has an additional risk that users voting power will be out of sync after `NFTBoostVault::setMultiplier`, 
until `_syncVotingPower` get triggered as part of a NFT update or the bulk update the NFT multiplier update will not
have cascaded to all existing token holders with that NFT already delegated.



**Mechanism review**
The Council framework is all about governance, but has a few ideas at it's core, including committees, 
motivating participation in votes, optimistic grants, different security thresholds, customizable by function selector and address
and voting vaults that provide custom power calculations.

The Governance Steering Council (GSC), is a committee that has a fluid membership (determined by delegated voting power) 
and can create proposal without the spam threshold and has a limited discretionary fund at their disposal (`ArcadeGSCVault` and `ArcadeGSCCoreVoting`).

Optimistic grants angle is fulfilled by `ARCDVestingVault`, where vesting grants are created and may be revoked, with the
`ImmutableVestingVault` having the revocation option removed.

Custom power calculations are provided by the `NFTBoostVault` that uses a multiplier based on the NFT locked by the user.
The idea of community participation or reward badges (`ReputationBadge` and `BadgeDescriptor`) that can increase the
governance voting power that a user can delegate (even to themselves).



**Systemic risks**
Smart contract risk: as the external dependencies are on other smart contracts there exists risks in their implementations (possible bugs, maybe exploits)

Populism: voting power being determined by amount of locked tokens and NFT multiplier with the ability to delegate, 
could result in demagoguery that acts against for the benefit of the few at the expense of the protocol.

Vigilance apathy: without any hard limit on the number of proposals that may be submitted through the GSC and general 
process, the volume may become sufficient to cause apathy, where objective and vigilant reviewers disengage and valid 
criticisms of corruption or centralisation of voting power through multiplier manipulation are not raised.


### Time spent:
25 hours