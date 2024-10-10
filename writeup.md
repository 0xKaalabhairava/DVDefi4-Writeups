# Unstoppable CTF Writeup

## Challenge Description
There's a tokenized vault with a million DVT tokens deposited. It’s offering flash loans for free, until the grace period ends.

To catch any bugs before going 100% permissionless, the developers decided to run a live beta in testnet. There's a monitoring contract to check liveness of the flashloan feature.

Starting with 10 DVT tokens in balance, show that it's possible to halt the vault. It must stop offering flash loans.


## The Exploit Strategy
There are two contracts called UnstoppableMonitor.sol and UnstoppableVault.sol. in UnstoppableVault.sol there is a vuln in function `flashLoan`. The line `(convertToShares(totalSupply) != balanceBefore)` enforces a condition that requires the totalSupply of vault tokens to always match the totalAssets of underlying tokens before any flash loan execution. This condition acts as a safeguard to ensure that the flashLoan function remains inactive if the vault deviates assets to other contracts.

The attack strategy revolves around creating a conflict between the two distinct accounting systems. This is achieved by manually transferring DVT tokens directly to the vault.

This manipulation disrupts the balance and leads to a scenario where the condition `(convertToShares(totalSupply) != balanceBefore)` fails.

Consequently, the flashLoan function is deactivated due to the divergence in accounting systems, since the transaction will always be reverted without dependance on the “user” input.

## Solution
```solidity
    function test_unstoppable() public checkSolvedByPlayer {
        require(token.transfer(address(vault), 1));   
    }
```

## References
- [Solution Link](https://docs.soliditylang.org/)
- [Damn Vulnerable DeFi](https://www.damnvulnerabledefi.xyz/)
