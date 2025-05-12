# Token


The goal of this level is for you to hack the basic token contract below.

You are given 20 tokens to start with and you will beat the level if you somehow manage to get your hands on any additional tokens. Preferably a very large amount of tokens.

  Things that might help:

What is an odometer?

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {
    mapping(address => uint256) balances;
    uint256 public totalSupply;

    constructor(uint256 _initialSupply) public {
        balances[msg.sender] = totalSupply = _initialSupply;
    }

    function transfer(address _to, uint256 _value) public returns (bool) {
        require(balances[msg.sender] - _value >= 0);
        balances[msg.sender] -= _value;
        balances[_to] += _value;
        return true;
    }

    function balanceOf(address _owner) public view returns (uint256 balance) {
        return balances[_owner];
    }
}
```
## The Problem: Arithmetic Overflow/Underflow
- Here, you are checking if the sender's balance after subtracting the _value is non-negative. While this looks good at first glance, it does not properly protect against arithmetic underflows.

# What’s the issue?
- Underflow: If balances[msg.sender] is smaller than _value (e.g., if the balance is 10 and the user tries to transfer 20), subtracting the values will result in an underflow, meaning that the result will wrap around to a very large number (because of the way integers work in most programming languages).
- No Check for Overflow/Underflow: In Solidity versions prior to 0.8.0, this check doesn't prevent the underflow from occurring because the subtraction could result in a number less than zero, which wraps to a large positive number, making the check useless.

## For example:

If the sender has balances[msg.sender] = 10, and the user tries to transfer _value = 20, without proper checking, the subtraction will underflow, and instead of -10, the result will be a large positive number due to integer wraparound (essentially 2^256 - 10).
- If the balances[msg.sender] is smaller than the _value being transferred, the subtraction (- _value) will underflow and cause the balance to wrap around to a very large number (due to how unsigned integers work in Solidity).

## Exploitation:
```javascript
await contract.transfer(address_account, 30)
```
- My balance is 20. So, I will transfer 30
- Subtraction results in a bug, thus, an underflow happens and my balance explodes :/ :D


## Prevention: How to Fix the Vulnerability
- To prevent this from happening, you need to ensure that the subtraction is safe and doesn’t allow underflows. As mentioned earlier, Solidity 0.8.0 and later automatically revert transactions that would cause overflow/underflow.
-For older versions, you should use a library like SafeMath to safely handle arithmetic operations.
