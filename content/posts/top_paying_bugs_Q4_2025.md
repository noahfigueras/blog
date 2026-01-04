---
title: "Top Paying Bugs of Sherlock Q4 2025"
date: 2025-11-21
slug: "top-paying-bugs-q4-2025"
description: "I have analyzed the top paying findings for Q4 2025 in sherlock"
keywords: ["security", "solidity"]
draft: false
tags: ["security", "solidity"]
---

# Analyzing Sherlock Q4 top paying bugs.

I will be analyzing here bugs that paid over 200$ as that is at least what I would 
want to get paid for a bug in competition to make this profitable. 


| Contest | Issue | Description | Reward |
|---------|-------|-------------|--------|
| [Index Fun Order Book](https://audits.sherlock.xyz/contests/1197) | M - Asymmetric fee structure allows market participants to get the same outcome for less fee | Users can take different paths to achieve lesser fees in trades. | $5492 |
| | M - Violation of rule defined in EIP-1155 | “The URI MUST point to a JSON file that conforms to the ERC-1155 Metadata URI JSON Schema.” However, in the current implementation, the URI is set to an empty string, violating this requirement. | $720 |
| | M - Lack of Emergency Market Invalidation Mechanism | Lack of Mechanism, Market has no way to resolve to some state. | $720 |
| [Summer.fi Governance V2](https://audits.sherlock.xyz/contests/1176) | M - Foundation recall will inflate governance power for earnest voters | Missing logic to reduce escrowed weights after recallUnvestedTokens will cause an enduring governance power inflation for honest voters. | $381 |
| | M - Satellite chains can't execute onlyGovernance functions | The root cause is that the satellite chain execution logic fails to properly adapt to the standard OpenZeppelin (OZ) Governor design. | $5668 |
| [Super DCA Liquidity Network](https://audits.sherlock.xyz/contests/1171) | M - Attackers will steal rewards from legitimate pools by making duplicate pools for listed token | Lack of pool-specific validation in `_handleDistributionAndSettlement` to check if the pool is listed before accruing rewards. Use of Uniswap V4 Hooks. | 232 OP |
| | H - Unfair distribution of rewards for LPs | The pool's reward distribution occurs during each liquidity operation through the beforeAddLiquidity and beforeRemoveLiquidity hooks. The Uniswap pool's donate mechanism is used, which distributes rewards to in-range LPs at the current slot0.tick. The problem with this design is that rewards accrued over a certain period are distributed to the current in-range position, regardless of which positions provided active liquidity during the accrual period. | 5661 OP |
| [Dango DEX](https://audits.sherlock.xyz/contests/1066) | H - Incorrect rounding direction in geometric pool ask_exact_amount_out allows theft of funds| | 7467 $|
| | H - Exploitation of fee bypass through Deposit-Then-Withdraw strategy | $4480 |
| | M - User can pause all auctions by overflow in mid-price average causing DOS | $3318 |
| | M - Overflow in geometric can dos swap | $3318 |
| | M - Price cannot be represented when a high-value base asset is quoted in a high-decimal asset | $3318 |
| | M - Inconsistent multiplication during order creation and cancellation can lead to panics and Denial of Service during order cancellation | $3318 |
| | H - Geometric pool swap costs can be evaded through repeated small transactions leading to a loss of yield for LP's | $1633 |
| | M - XYK reflect_curve omits swap fee in order sizing, leaking LP fees | $1493 |
| | M - XYK reflect_curve is incorrect and decreases K over time leading to loss of funds for LP providers. | $1493 |
| | M - Unbounded spam limit asks can be created to DOS force cancellation | $1493 |
| [Brevis Pico ZKVM](https://audits.sherlock.xyz/contests/873) | | | |
| [Ammplify](https://audits.sherlock.xyz/contests/1054) | | | |
