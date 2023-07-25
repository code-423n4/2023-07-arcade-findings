## Centralization Risk

- owner of CoreVoting can set arbitrary votingVault to true; which adds up the totalPower of queryVotePower arbitrary; this adds risk to decentralisation, that means any address can be added that implement queryVotePower to returns type(uint256).max

- The owner can also set arbitrary quorum, minVotingPower etc; that means it can within a block, setminVotingPower to 0, quorum to 0, propose, execute proposal at the same time, and reset the voting power and quorum back to the origin value.


## Code Quality
Overall the code quality is very high, with known issues and expected behavior added on top of function comments which are highly appreciatedd. Whenever possible, the team has made an effort to solidified variables and using immutable to secure the code.

The team has also to keep some flexibility, for example to GSC actor, to enact small and operational tasks, while keeping the some of the major decisions to be dao-driven. This is very appreciated too.


### Time spent:
4 hours