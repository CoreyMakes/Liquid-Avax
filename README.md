---
description: >-
  This is our subscription to Avalanche developers contest. We have chosen the
  "How to create staking derivatives on Avalanche" subject.
---

# Liquid Avax

## What is liquid staking ? 

Liquid staking is an alternative to staking. The advantage of liquid staking is that staked asset are not locked anymore, while still earning returns and supporting decentralisation.



## How does it work ?



Using the interface, the user is able to stake its $AVAX and get $sAVAX in exchange at a 1:1 rate. The user can also monitor all its current stakes if the stake is not finished yet and/or not withdrew by the backend. To withdraw the funds, the user will need to provide back the $sAVAX tokens against what he will get his $AVAX again with the staking rewards.



## Let's present the project more in depth



This project is composed of several components:

1. Frontend
2. Smart Contract
3. Backend

## Frontend



The frontend, or user interface is our platform called "Liquid Avax". We forked Pancake Swap for the sake of simplicity, and security.



## Smart Contract



The smart contract is based on ERC20 smart contracts. It handles requests made by the frontend, and emits events for the backend.

## Backend



The backend is basically a nodeJS script running 24/7 waiting for an Event such as "Staked" or "StakeEnded" to be emitted from the smart contract.

When the event "Staked" is emitted, it will retrieve the staker address, the stakeID, and the amount of $AVAX staked from it. This will be used to transfer the funds from the C to P chain and stake the funds on a node for a defined amount of time.

When the event "StakeEnded" is detected, the user will be able to redeem the funds. The backend will cross chain from P to C and send the funds to the smart contract.

