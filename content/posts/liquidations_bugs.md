---
title: "A primer on DeFi Liquidations and how can they be compromised."
date: 2026-01-04
slug: "web3_security"
description: "Analysis of common vulnerabilities in Lending Protocols: Liquidations."
keywords: ["security", "solidity", "defi", "liquidations"]
draft: false 
tags: ["security", "solidity"]
---

# Introduction
During the past month or so, I've been researching aave new v4 implementation during
a contest in [sherlock](https://audits.sherlock.xyz/contests/1209).

My focus diving deep into the protocol was not only to focus on finding bugs, but
to explore his new implementation and learn how it works for future implementations.

Today, I wanted to write a bit on the topic of Liquidations: what they are, how they
work and some attack vectors to keep in mind while auditing.

Liquidations are a core mechanism for any Lending protocol, so this knowledge is not 
really specific to aave protocol, it applies to any DeFi protocol utilizing
lending and borrowing capabilities.

Without further introduction, let's dive deep into the topic and digest ways to 
exploit them.

## What are Liquidations and how they work
Liquidations are an essentiall utility to fix bad debt in any lending protocol. 
Users commonly have 4 types of operations in a Lending protocol: 

1. **Supply**: Users supply some tokens with a promise to get an x% interest annually. 
2. **Borrow**: Users borrow some tokens utilizing at least x% collateralFactor to protect
their position from bad debt.
3. **Repay**: Users need to repay their debt with %x interest accumulated in return. 
4. **Liquidations**: Liquidators are in charge to close borrowing positions that are 
in debt beyond the collateral provided.

To clarify, the term `collateralFactor` represents a % of how much borrowing tokens
a user can borrow with `x` collateral. The `collateralFactor` is usually a configurated
value that can change for different tokens depending on their risk or volatility.

Let's say `alice` wants to borrow `1 WETH` tokens with `USDC` as collateral and 
the collateral Factor to borrow `ETH` is `80%`. Then assuming `1 WETH == 2000 USDC` , 
the minimum collateral that `alice` needs to provide to get `1 WETH` in `USDC` 
will be `2500 USDC` so: 

```bash
                           LOAN_VALUE
MIN_COLLATERAL_VALUE = ----------------
                        COLLATERAL_FACTOR
```

Without the minimum requirements of collateral provided, a user should not be able
to borrow any tokens, if it does it is a bug. But, we'll talk more about attack 
vectors later on.

Now, we understand the **collateral factor** or often called **LTV** (Loan to Value)
which determines your **maximum borrowing power** based on your collateral's **USD value**. 

After a loan is created, the price of both borrowing and collateral tokens can 
change due to market volatility and this can heavily influence the state of the 
loan. That's exactly why liquidations come into place, in order to protect the protocol
from users leaving bad debt and cause protocol insolvencies.

The role of Liquidators then, is to monitor the protocol for borrowing positions 
that move beyond their provided **collateral value**. 

Following the previous example, if `WETH` moves in price to `3000 USDC`, then **alice**
previous position will have to be liquidated as her collateral can't cover all of 
her outstanding debt.

That introduces us to the **Health Factor**: the key metric used to calculate the 
**safety** of a borrower's position: 

```bash
     COLLATERAL_VALUE * LIQUIDATION_THRESHOLD
HF = ----------------------------------------
                BORROWED_VALUE
```

It is a single numerical value that tells you how close your deposited collateral 
is to being liquidated. The Health Factor operates on a simple rule:

- **HF => 1**: Your loan is considered **safe**.  
- **HF < 1**: Your loan is **eligible for liquidation**.

Example:  
Using `alice` previous position with a `LIQUIDATION_THRESHOLD = 83%` this will be 
the following: 

`2500 * 0.83 / 3000 = ~ 0.69` HF less than 1. This will cause the position to 
be liquidated. 

Now, that we know a primer on liquidations. Let's look at some attack vectors that 
we can exploit and some bugs that have been found in the past.

## Attack Vectors 

In principle, in most of the Liquidation engines there are some safety invariants
that need to hold.

1. Liquidators can't liquidate positions with a HF >= 1.
2. Liquidators can liquidate positions with a HF < 1.
3. Liquidators have to be incentivized for this, so they need to end in profit after liquidations.
4. The total value of the liquidated collateral (after the penalty/discount) must exactly match the amount of debt being repaid plus the liquidator's bonus.
5. The liquidation process must only seize collateral assets and repay borrowed assets that are supported by the protocol.

Any implementations violating any of this invariants are vulnerable to attacks.

Following are some attack vectors and examples from vulnerabilities find in the field: 


* Can a Liquidator liquidate a user position with the shares of his own position?  
If so, can the liquidator liquidate the position and become undercollateralized?

* Does the `liquidate()` allow a liquidator to liquidate a position with a HF < 1 
under all conditions?. [example](https://solodit.cyfrin.io/issues/liquidate-does-not-allow-the-liquidator-to-liquidate-a-user-if-the-liquidator-hf-1-codehawks-foundry-defi-stablecoin-codehawks-audit-contest-git)

* Does the `liquidate()` allow a liquidator to liquidate a position with a HF >= 1.

* Liquidation can not work when most of the liquidity is borrowed, causing a position
to not be liquidated. This issue can happen in protocols when the `transferFrom()`
of the liquidator debt repayment happens after collateral transfer in sequence.
[example](https://solodit.cyfrin.io/issues/m-11-marketliquidate-will-not-work-when-most-of-the-liquidity-is-borrowed-due-to-wrong-liquidator-transferfrom-order-sherlock-exactly-protocol-git)

* Incorrect liquidation fee calculation during underwater liquidations, this will
not incentivize liquidators to participate. If liquidations are not profitable this can cause the 
protocol to become insolvent.
[example1](https://solodit.cyfrin.io/issues/m-24-incorrect-liquidation-fee-calculation-during-underwater-liquidation-disincentivizing-liquidators-to-participate-code4rena-revert-lend-revert-lend-git)
[example2](https://solodit.cyfrin.io/issues/m-15-liquidators-will-not-be-incentivized-to-liquidate-certain-partyb-accounts-due-to-the-lack-of-incentives-sherlock-none-symmetrical-git)

* Liquidations not accounting for a penalty fee.
[example](https://solodit.cyfrin.io/issues/h-02-liquidation-doesnt-account-for-penalty-when-calculating-collateral-to-give-allowing-users-to-profit-by-borrowing-and-self-liquidating-code4rena-loopfi-loopfi-git)

* Liquidations not profitable for liquidators.
[example](https://solodit.cyfrin.io/issues/m-4-in-some-situations-the-liquidation-can-become-unprofitable-for-liquidators-keeping-unhealthy-positions-sherlock-aloe-git)

* Are liquidated positions able to be liquidated more than once ? 
[example](https://solodit.cyfrin.io/issues/m-10-quote-that-have-already-been-liquidated-can-be-liquidated-again-in-some-cases-sherlock-none-symmetrical-git)

* Front-Running Scenarios Can happen where a user can prevent others to liquidate
his position by front-running liquidations.
[example](https://solodit.cyfrin.io/issues/h-01-liquidations-can-be-prevented-by-frontrunning-and-liquidating-1-debt-or-more-due-to-wrong-assumption-in-pos_manager-code4rena-init-capital-init-capital-git)

* What happens with underwater positions, this are positions that the amount of 
debt has exceeded it's collateral. Can all of them be safely liquidated without 
causing insolvency for the protocol?

To conclude, Liquidations are an essential part of any Lending system and as hackers
we need to make sure we understand them and can find vulnerabilities that might be 
costly in the future. With this post I don't intend to give a full guide, but just 
a glimpse on the topic and my thinking behind Liquidations as a Solidity Auditor. 

I will welcome any feedback or contributions to improve this article. 

Happy Hacking!

`0xSolus`
