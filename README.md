---
description: >-
  This is our subscription to Avalanche developers contest. We have chosen the
  "How to create staking derivatives on Avalanche" subject.
---

# Liquid Avax

## Quick prensentation

#### What is liquid staking ?

Liquid staking is an alternative to staking. The advantage of liquid staking is that staked asset are not locked anymore, while still earning returns and supporting decentralisation.



#### How does it work ?

Using the interface, the user is able to stake his $AVAX and get $sAVAX in exchange at a 1:1 rate. The user can also monitor all its current stakes if the stake is not finished yet and/or not withdrew by the backend. To withdraw the funds, the user will need to provide back the $sAVAX tokens against what he will get his $AVAX again with the staking rewards.



### Let's present the project more in depth...

This project is composed of several components:

1. Frontend
2. Smart Contract
3. Backend

#### Frontend

The frontend, or user interface is our platform called "Liquid Avax". We forked Olive Swap, a fork of Pancake Swap for the sake of simplicity, and security on Avalanche.

#### Smart Contract

The smart contract is based on ERC20 smart contracts. It handles requests made by the frontend, and emits events for the backend.

#### Backend

The backend is basically a nodeJS script running 24/7 waiting for an Event such as "Staked" or "StakeEnded" to be emitted from the smart contract.

When the event "Staked" is emitted, it will retrieve the staker address, the stakeID, and the amount of $AVAX staked from it. This will be used to transfer the funds from the C to P chain and stake the funds on a node for a defined amount of time.

When the event "StakeEnded" is detected, the user will be able to redeem the funds. The backend will cross chain from P to C and send the funds to the smart contract.



## Tutorial

Okay, we hope that you are now full set to go through the tutorial. We will start by the smart contract.

All the code can be found on our github, please head on to [https://github.com/iFrostizz/Liquid-Avax/tree/master](https://github.com/iFrostizz/Liquid-Avax/tree/master)

### Smart contract



### Backend

Now that the smart contract is ready to go, let's prepare the backend in consequence!

I you want to use github, create a repository, clone it, and enable "VCS" if you want to keep track on your work. Otherwise, just create a directory that you will open in your IDE.

Now, we will setup a node development environment.

First, check if node is installed on your machine. To verify this, run

```text
node -v
```

If the output looks like this:

```text
v14.16.0
```

You don't need to follow the installation steps.

If not, please install node v12.0.0 or earlier.

#### Windows / macOS

1. Go to this link: [https://nodejs.org/en/](https://nodejs.org/en/)
2. Click on xx.xx.x LTS recommended for most users
3. Follow the procedure described after running the downloaded file

#### Ubuntu 20.4

```text
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
```

Please also verify that npm is properly installed, it should be bundled with node:

```text
npm -v
```

If the output is something that is similar to this:

```text
7.17.0
```

You're ready!

Start by creating an Avalanche wallet here: [https://wallet.avax.network/](https://wallet.avax.network/) by clicking on "Create new wallet", and "Generate key phrase". After that you accessed to your wallet, head on to "Manage keys", and "View C Chain private key".

{% hint style="info" %}
Make sure to never share this private key. Anyone can access to your wallet using it.
{% endhint %}

Copy and paste it in a ".json" file. Let's call it "privateData.json"



First, copy and paste the code skeleton. We will go through each step.



