1 . Target : https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol

- SafeMath

Issue: The contract does not use the SafeMath library to perform arithmetic operations on unsigned integers. This may lead to potential arithmetic overflows and underflows, which can result in unintended behavior or even security vulnerabilities.

Proof-of-Concept: Let's consider an example of an arithmetic overflow. Suppose the contract has a spend threshold of 100, and an attacker tries to spend 2^256 - 1 (maximum uint256 value) tokens in a single transaction.

// Incorrect Implementation
uint256 threshold = 100;
uint256 amount = uint256(-1); // 2^256 - 1 (maximum uint256 value)
uint256 newThreshold = threshold + amount;

Impact: The arithmetic overflow can cause the newThreshold variable to wrap around and have a value much smaller than expected. This could lead to unintended consequences, such as reducing the threshold to a very low value, effectively bypassing spending restrictions.

Recommendation: Use the SafeMath library to perform arithmetic operations on unsigned integers to prevent overflows and underflows.

// Recommended Implementation
import "@openzeppelin/contracts/utils/math/SafeMath.sol";

...

using SafeMath for uint256;

uint256 threshold = 100;
uint256 amount = uint256(-1); // 2^256 - 1 (maximum uint256 value)
uint256 newThreshold = threshold.add(amount);



- Ether Transfer Safety

Issue: In the _spend function, the contract uses the .transfer function to send Ether to the destination address. The .transfer function has a gas stipend of 2,300, which may not be sufficient for complex contract interactions. This can lead to a "revert without a reason" error if the recipient is a contract that requires more gas.

Proof-of-Concept: Suppose the recipient is a contract with a fallback function that consumes more gas than the 2,300 stipend.

// Incorrect Implementation
function _spendEther(address payable destination, uint256 amount) internal {
    destination.transfer(amount);
}

Impact: If the recipient contract requires more gas for processing (e.g., updating storage, emitting events, etc.), the .transfer function will revert without a reason, causing funds to remain in the treasury and leading to unexpected behavior for users.

Recommendation: Use the .call{value: amount}("") pattern to send Ether to the recipient, allowing more gas for contract interactions.

// Recommended Implementation
function _spendEther(address payable destination, uint256 amount) internal {
    (bool success, ) = destination.call{value: amount}("");
    require(success, "Ether transfer failed");
}



- Approve and Call

Issue: The contract uses IERC20(token).approve(spender, amount); to approve tokens for spending by the spender. This pattern is susceptible to a known attack called the "approve and call" attack, where an approved contract can execute an arbitrary function using the approved tokens.

Proof-of-Concept: Suppose there is a malicious contract MaliciousSpender:
// Malicious Contract
contract MaliciousSpender {
    address public treasury;

    constructor(address _treasury) {
        treasury = _treasury;
    }

    function receiveApproval(address _from, uint256 _amount, address _token, bytes memory _data) external {
        // Malicious function
        // Do something malicious with the approved tokens
    }
}


Impact: If the MaliciousSpender contract is approved to spend tokens from the treasury, it can execute arbitrary code in the receiveApproval function and potentially harm the treasury or users.

Recommendation: Consider using the "pull" pattern or more robust token approval mechanisms like approve and transferFrom in a separate step. For example, use a two-step process where users first approve a certain amount of tokens to be spent by the contract and then trigger the contract function to transfer those tokens.

// Recommended Implementation
function approveTokens(address token, uint256 amount) external {
    IERC20(token).approve(address(this), amount);
}

function spendTokens(address token, uint256 amount, address destination) external {
    IERC20(token).transferFrom(msg.sender, destination, amount);
}





