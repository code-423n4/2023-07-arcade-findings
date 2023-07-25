There are a number of places in the NFT Boost Vault where a check is made to ensure that for a token to have the multiplier effect, the token id must be non zero. E.g 
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L422-L423. There is also no provision that ReputationBadge contract (https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol) preventing a token with `token id == 0` from being minted. This means that the owner of that token will never be able to get an nft boost.

A suggested solution will be to base disqualification from boosting on the condition that `tokenAddress == 0` instead of using both the tokenAddress and tokenId

Also, calling the `delegate` function in the the ARC vesting vault can lead to a false sense of security of delegation as it would be successful even if the sender has no grant. It is particularly bad because they might also think that after being given a grant on a future date, the delegate value will not change which is wrong as it is overridden by whatever is specified or it will their own address if none is specified. Apart from that, you could delegate to a zero address and it would be successful.

It is recommended that there should be a check to prevent delegation to zero addresses and delegation should not be successful unless there was a previous allocation.
