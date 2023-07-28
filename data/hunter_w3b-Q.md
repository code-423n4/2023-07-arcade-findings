## [L-01] Lack of Chain `Identification` in NFT Contract

The NFT contract located at contracts/nft/BadgeDescriptor.sol does not handle hardforks and lacks proper chain identification. To prevent confusion about which chain is the owner of the NFT, it is essential to include the chain ID in the functions or URIs.

To address this issue, we recommend modifying the tokenURI function to include the chain ID in the URI. This can be achieved by using require(1 == chain.chainId) or by incorporating the chain ID in the returned URI string.

Updated tokenURI Function:

```solidity

function tokenURI(uint256 tokenId) external view override returns (string memory) {
    uint256 chainId = chain.chainId; // Assuming there's an external reference to obtain chain ID.
    return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "chain-", chainId.toString(), "/", tokenId.toString())) : "";
}

```

Alternatively, if the contract has access to the chain ID directly without external references, you can use the following code:

```solidity

function tokenURI(uint256 tokenId) external view override returns (string memory) {
    uint256 chainId = block.chainid; // Accessing chain ID directly from the block object.
    return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "chain-", chainId.toString(), "/", tokenId.toString())) : "";
}

```

By incorporating the chain ID in the URI, this modification will ensure clarity about the NFT's ownership on a specific chain and mitigate potential confusion during hardforks.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L48-L50

## [L-02] Potential `DOS` Vulnerability in Unbounded For-Loop

The contract contracts/ArcadeTreasury.sol contains a potential Denial of Service (DOS) vulnerability due to an unbounded for-loop making external calls. Unbounded loops that make external calls can lead to excessive gas consumption and may result in a DOS attack.

To address this issue, it is crucial to limit the number of iterations in the for-loop that makes external calls. This can be achieved by carefully assessing the maximum number of iterations required for the specific use case and incorporating a check to prevent excessive looping.

```solidity
function executeTargets(address[] memory targets, bytes[] memory calldatas) external {
    require(targets.length == calldatas.length, "Array lengths do not match");

    uint256 maxIterations = 10; // Set an appropriate maximum number of iterations.

    // Limit the loop to the minimum of the actual array length and the maxIterations.
    uint256 loopLength = targets.length < maxIterations ? targets.length : maxIterations;

    for (uint256 i = 0; i < loopLength; i++) {
        (bool success, ) = targets[i].call(calldatas[i]);
        require(success, "External call failed");
    }
}
```

By introducing a limit on the number of iterations, the contract can mitigate the risk of a potential DOS attack by restricting the gas consumption of the loop and maintaining better control over its execution. Please adjust the value of maxIterations according to the specific requirements of your use case to strike a balance between functionality and security.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L341

## [L-03] `Gas Griefing` Vulnerability in Unsafe External Call

The contract contracts/ArcadeTreasury.sol contains a gas griefing/theft vulnerability due to an unsafe external call. The external call made in the for-loop at line 341 does not specify a limit on the amount of gas used. As a result, the recipient of the call can consume all of the transaction's gas, causing it to revert and potentially leading to a loss of funds.

To address this issue and prevent gas griefing or theft, you should use a more secure method of executing external calls, such as call.gas(), which allows you to set a specific amount of gas to be sent with the call. This way, you can control the maximum amount of gas that can be consumed during the call.

Example of using `call.gas()`:

```solidity

function executeTargets(address[] memory targets, bytes[] memory calldatas, uint256 gasLimit) external {
    require(targets.length == calldatas.length, "Array lengths do not match");

    uint256 maxIterations = 10; // Set an appropriate maximum number of iterations.

    // Limit the loop to the minimum of the actual array length and the maxIterations.
    uint256 loopLength = targets.length < maxIterations ? targets.length : maxIterations;

    for (uint256 i = 0; i < loopLength; i++) {
        // Set a gas limit for the external call to prevent excessive gas consumption.
        (bool success, ) = targets[i].call{gas: gasLimit}(calldatas[i]);
        require(success, "External call failed");
    }
}
```

By specifying a gas limit for the external call, the contract can protect against gas griefing attacks and ensure that the transaction remains within reasonable gas consumption bounds. The gasLimit parameter should be carefully chosen based on the expected gas requirements of the target functions and any potential gas price fluctuations. It is essential to strike a balance between providing sufficient gas for the external call to succeed and preventing excessive gas consumption.
