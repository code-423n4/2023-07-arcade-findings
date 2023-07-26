Missing check for matching array lengths

To avoid a revert during execution, the following functions should check for matching array lengths ie

- contracts/external/council/voting/CoreVoting.sol vote:Ln 215 parameters votingVaults and extraVaultData.

- contracts/external/council/vaults/GSCVault.sol/proveMembership vote:Ln 56 parameters votingVaults and extraData.

- contracts/external/council/vaults/GSCVault.sol/kick vote:Ln 110 parameters votingVaults and extraData.

- contracts/ArcadeTreasury.sol/batchCalls::Ln333 parameters targets and calldatas.