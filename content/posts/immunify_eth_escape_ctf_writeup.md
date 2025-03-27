---
title: "Immunify ETH ESCAPE CTF Writeup"
date: 2024-12-01
slug: "ctf writep"
description: "This are some of the solutions for the Immunify ETH ESCAPE CTF"
keywords: ["security", "solidity", "ctf"]
draft: false
tags: ["security", "solidity", "ctf"]
summary: This are some of the solutions for the Immunify ETH ESCAPE CTF.
---

# Immunify ETH ESCAPE CTF Writeup 

During devcon 7 in bangok, I participated in ETH ESCAPE ctf organized by immunify. 
It was my first experience, playing an on-site ctf with time restriction. In this, 
contest the time was 50min. I did really bad, so this is me trying to review and 
solve some of the challenges after the contest. 

Check all the problems and solutions in my [github](https://github.com/noahfigueras).


## Sekai
This was an easy challenge that required some knowledge about how mappings are 
stored in the evm. 

We have to find the key stored in `keys[0xdead]` in the constructor to `unlockSekai()`
and solve the challenge. 

In order to do this, we need to retrieve the slot where the key is saved. In the evm, mappings 
are stored at slot `keccak256(abi.encode(x, y))` being `y` the slot of the mapping 
variable `keys` which in this case is 1 and `x` the key of the mapping `0xdead`.

Solution:  
```
  function test_Sekai() public {
    bytes32 slot = keccak256(abi.encode(uint256(0xdead), uint256(1)));
    sekai.unlockSekai(uint256(slot));
    assertEq(sekai.isKeyFound(), true);
  }

```

## Curve Validator
In order to solve this challenge we need to have a basic understanding of what are 
elliptic curves and how we can perform point addition.

```
https://blog.cloudflare.com/a-relatively-easy-to-understand-primer-on-elliptic-curve-cryptography/
```

The elliptic curve is setup with some constants `A` and `B` and a finite field `p`.
The formula of the elliptic curve is `y**2 = x**3 + ax + b mod p`. If that equation resolves, 
then we have a point `P(x,y)` on the curve. 

In order to pass `validate(x2,y2)` we have to ensure that the points passed are on the 
curve and also the point addition of `P(W0x, W0y) + Q(x2, y2) = R(X[0], y3)`. 
```
https://www.rareskills.io/post/elliptic-curve-addition
```
At this point, we have points `P` and `R`(as we can deduct `y3` from `X[0]`), all we have 
to do is a bit of magic to find out `x2`.

```
The point addition formula for elliptic curves rearanged for x2 is as follow:

x**2 = λ**2 -x1 - x3 mod p 
λ = y3 + y1 / x3 - x1 mod p 

```
Solution:  
```solidity
  function test_CurveValidator() public {
    assert(p%4 == 3);
    uint256 x1 = W0x;
    uint256 y1 = W0y;
    uint256 x3 = X[0];
    uint256 y3 = modExp((x3**3 + A*x3 + B), (p + 1) / 4, p);

    uint256 lambda = ((y3 + y1) * modInverse((p - (x1 - x3)))) % p;
    uint x2 = (lambda * lambda - x1 - x3) % p;
    uint y2 = modExp((x2**3 + A*x2 + B), (p + 1) / 4, p);
    curveValidator.validate(x2, y2);
    assertEq(curveValidator.isSolved(), true);
  }
```

## Curve Validator Plus
TODO: 
