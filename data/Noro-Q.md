# Risk of silent overflow
- `ARCDVestingVault.revokeGrant()` ,`ARCDVestingVault.claim()` updates the grant.withdrawn variable and `NFTBoostVault.delegate()` , `NFTBoostVault_syncVotingPower()` updates registration.latestVotingPower. However, these variable are of type `uint128`, while the values that update them are of type `uint256`. This means that casting to a lower type is necessary, but this casting is performed without first checking that the values being cast can fit into the lower type. As a result, there is a risk of a silent overflow occurring during the casting process.
# `call()` should be used instead of `transfer()` on an address payable
- The **`transfer()`** and **`send()`** functions forward a fixed amount of 2300 gas. Historically, it has often been recommended to use these functions for value transfers to guard against reentrancy attacks. However, the gas cost of EVM instructions may change significantly during hard forks which may break already deployed contract systems that make fixed assumptions about gas costs. For example. EIP 1884 broke several existing smart contracts due to a cost increase of the SLOAD instruction.
# critical functions doesn't log emit events
- BaseVotingVault.setTimelock()
- BaseVotingVault.setManager()
- ArcadeTokenDistributor.setToken()
- ArcadeAirdrop.reclaim()
- ARCDVestingVault.deposit()
- ARCDVestingVault.withdraw()

these functions play a central role in the protocol, but they don't emit any event, however, by emitting events, contracts can efficiently inform external entities about significant occurrences, enhancing the usability and transparency of decentralized applications and they also offer cost-efficient and permanent logs of essential state changes .