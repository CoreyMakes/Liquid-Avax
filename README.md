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

Using the interface, the user is able to stake his $AVAX and get $sAVAX in exchange at a 1:1 rate. The user can also monitor all its current stakes if the stake is not finished yet and/or not withdrew by the backend. To redeem the funds, the user will need to provide back the $sAVAX tokens against what he will get his $AVAX again with the staking rewards.



### Let's present the project more in depth...

This project is composed of several components:

1. Frontend
2. Smart Contract
3. Backend

#### Frontend

The frontend, or user interface is our platform called "Liquid Avax". We forked Olive Swap, a fork of Pancake Swap for the sake of simplicity, and security on Avalanche.

#### Smart Contract

The smart contract is based on ERC20 smart contracts. It handles requests made by the frontend, and emits events for the backend.

We start by importing the basic ERC20 smart contract (this is a common standart for creating tokens, more details here :) and the Ownable one (this is a contract wich permit to restrain function to an owner adress).
```text
pragma solidity ^0.8.0;

import "ERC20.sol";
import "Ownable.sol";
```

Then we need to write 4 main functions : 
- One when a user deposit AVAX and mint sAVAX
- One to permit the backend to withdraw this funds
- One for the backend to put back fund + rewards on the contract
- And one to permit the user to get their funds back in exchange of sAVAX

In addition for clarity and reliability we will also need :
- One function to make the P-Chain tx viewable on C-chain (updated by the backend)
- One view function to view all stakes by an user
- One function to update some variables (as minimum staking time etc when future gouvernance changes it)

We will also need some variables to stock the staking informations :
We willuse a struct to store all informations relative to a stake :
```text
struct Stake{
    address payable user;
    uint256 stakeId ;
    uint256 amount ;
    uint256 finalAmount ;
    uint256 endingTimestamp ;
    bool withdraw ;
    bool updated ;
    bool redeemed ;
    string PChainTx ;
}
```
and we will use a mapping to link an id to a stake and a int containing the number of stake:
```text
uint256 public stakeNumber = 0;
mapping (uint256 => Stake) public stakeds ;
```

To make frontend call easier we will also link an adress with all his stakes ids :
```text
mapping(address=>uint256[]) public currentStakedByUser ;
```
and some variables so we can change this values later in case of gouvernance votes :
```text
uint256 public secondsInFuture = 14*24*3600 ;
uint256 public maxSecondsInFuture = 365*24*3600;
uint256 public secondsBeforeAllRedeem = 60*24*3600 ;
uint256 public minimumValue = 25 ether ;
```

Now let's begin with our first fonction to allow users deposit a stake.
This will just need 2 variables : the staking time (represented by a timestamp of the stake end) and the amount which is provided via msg.value in solidity
We just need some checking (the staking time should be greater than 2 weeks and smaller than 1 year / the staking amount should be larger than 25 AVAX)

We create a new Stake element and we populate his attribute with known ones, we map it to retrieve it with his id, we add it to the list of stakes by user and we can now mint some fresh sAVAX to the user adress and increase the number of stakes.
(we also emit an event which we will explain later)
```text
function stake(uint256 timestamp) public payable {
    require(timestamp > block.timestamp + secondsInFuture, "Ending period not enough in the future");
    require(msg.value >= minimumValue, "Not enough avax for delegation");
    Stake storage s = stakeds[stakeNumber];
    s.stakeId = stakeNumber;
    s.amount = msg.value ;
    s.endingTimestamp = timestamp ;
    s.user = payable(msg.sender) ;
    currentStakedByUser[msg.sender].push(stakeNumber);
    _mint(msg.sender, msg.value);

    emit Staked(s.user, s.stakeId, msg.value);
    stakeNumber++;
}
```

Now let's code the function allowing the backend to withdraw the funds in order to stake it :
```text
function withdraw(uint256 stakeId) public onlyOwner {
    Stake storage s = stakeds[stakeId] ;
    require(s.withdraw == false, "Already Withdrawn");
    require(payable(msg.sender).send(s.amount), "Send failed");
    s.withdraw = true ;
}
```
This function is very simple, it requires the caller to be the owner (aka backend private key), then it checks if the backend did not alreayd withdrawn funds for that stake and then send them to the back end adress

Now let's code the function which just update the PChainTx attribute of a stake :
```text
function setPChainTx(uint256 stakeId, string calldata h) public onlyOwner {
    Stake storage s = stakeds[stakeId] ;
    s.PChainTx = h ;
}
```
again very simple we just need to require it to be called by the owner then we update the PChainTx attribute.



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

If not, please install node v14.16.0 or earlier.

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

Copy and paste it in a ".json" file. Let's call it "privateData.json". If you plan to publish to your repo, create a .gitignore containing all the private stuff.

```text
{
  "privateKey": "D0N075H4r37H15704NY0N3"
}
```

Create a new file, we can call it backend.js .

Now, let's initiate the NodeJS development environment.

```text
npm init
```

Follow the prompt.

As we will import packages as ES6 modules, add this in the package.json :

```text
"type": "module",
```

Now, you can copy/paste the code skeleton in the .js file

```text
// This script will be used to do the cross-chain transfer of the owner wallet from C to P chain and stake them
// to a node.

import fs from "fs";
import avalanche from "avalanche";
import Web3 from "web3";
import { Buffer } from "buffer";

let binTools = avalanche.BinTools.getInstance();

let ip, protocol, port, networkID, avalancheInstance;

let xChain, xKeyChain, xChainAddress;
let cChain, cKeyChain, cChainAddress;
let pChain, pKeyChain, pChainAddress;

let xChainBlockchainID, cChainBlockchainID, pChainBlockchainID;

let web3;

let utxoset;

async function CtoP() { //a C --> P cross-chain transfer doesn't exists, but C --> X, X --> P does.

}

async function importKeys() { //Importing your encoded private key here to the node!

}

let transactionStatus;

async function waitForStatusX(transactionId) {
    transactionStatus = await xChain.getTxStatus(transactionId);
    return transactionStatus;
}

async function waitForStatusP(transactionId) { //Either X or P chain.
    transactionStatus = (await pChain.getTxStatus(transactionId)).status;
    return transactionStatus;
}

async function waitForStatusC(transactionId) {
    transactionStatus = await cChain.getAtomicTxStatus(transactionId);
    return transactionStatus;
}

async function pollTransaction(func, transactionID, resolve, reject) {
    transactionStatus = await func(transactionID);
    if (transactionStatus === "Rejected") {
        reject(transactionStatus);
    }
    if (transactionStatus !== "Accepted") {
        setTimeout(pollTransaction, 100, func, transactionID, resolve, reject);
    } else {
        resolve(transactionStatus);
    }
}

let xAvaxAssetId, cAvaxAssetId;

async function getInformations() { //Getting some relevant data for the code

}

async function setup() {
    await getInformations();
    await importKeys();
}

async function main() {
    await setup()
    await CtoP();
}

main().catch((e) => {
  console.log("We got an error! " + e);
})
```

Notice the X-Chain, C-Chain, P-Chain in the code ?

The Avalanche network operates 3 chains. 

* The X-Chain, or eXchange chain is the chain that is implementing the AVM \(Avalanche virtual machine\) with the Avalanche consensus. It is usually used to create, exchange assets.
* The C-Chain, or Contract chain is an instance of the EVM \(Ethereum virtual machine\) that is able to run smart contracts.
* The P-Chain, or Platform chain is implementing the snowman consensus, like the C-chain. It can be used to add a validator node \(A node that validates transactions on the network to keep it secure\), a delegator \(an actor staking his assets on validator nodes to help securise the network, while earning returns\), create custom chains, ...

You may have noticed that we imported some packages at the top of the code. Here is how to install a package and store it in the package.json:

```text
npm install <package>
```

And this how to install AvalancheJS for instance:

```text
npm install avalanche
```

Do that for each used package at the top of the code.

The "fs" package is used to manipulate files. In this case, it is our solution to not hard code the private key and keep it safe.

AvalancheJS is the Javascript library from Ava labs to interact with an Avalanche node. You can find their repository here with relevant code examples. [https://github.com/ava-labs/avalanchejs](https://github.com/ava-labs/avalanchejs)

The "web3" package will be used to interact with our smart contract.

Paste this in the `getInformations` function:

```text
ip = "localhost";
port = 9650;
protocol = "http";
networkID = 5;
avalancheInstance = new avalanche.Avalanche(ip, port, protocol, networkID);
    
const endpoint = '/ext/bc/C/rpc';
    
xChain = avalancheInstance.XChain();
xKeyChain = xChain.keyChain();
xChainBlockchainID = xChain.getBlockchainID();
    
cChain = avalancheInstance.CChain();
cKeyChain = cChain.keyChain();
cChainBlockchainID = cChain.getBlockchainID();
cChainAddress = cKeyChain.getAddressStrings();
    
pChain = avalancheInstance.PChain();
pKeyChain = pChain.keyChain();
pChainBlockchainID = pChain.getBlockchainID();
pChainAddress = pKeyChain.getAddressStrings();
    
xAvaxAssetId = binTools.cb58Encode((await xChain.getAssetDescription("AVAX")).assetID);
cAvaxAssetId = binTools.cb58Encode((await cChain.getAssetDescription("AVAX")).assetID);
    
web3 = new Web3(`${protocol}://${ip}:${port}${endpoint}`);
```

This is the place where we are instanciating our nodes for both AvalancheJS and web3. We are also storing some data that will be useful for the future.

{% hint style="info" %}
These values are valid for a local node on fuji only.
{% endhint %}

This will be the content of your `importKeys` function:

```text
const pKeyHex = JSON.parse(await fs.readFileSync("./data.json")).privateKey;
let buffer = Buffer.from(pKeyHex, 'hex')
let CB58Encoded = `PrivateKey-${binTools.cb58Encode(buffer)}`
xKeyChain.importKey(CB58Encoded);
cKeyChain.importKey(buffer);
pKeyChain.importKey(CB58Encoded);
```

We are getting the private key from our file and converting it to buffer for the C-chain. For the X and P-Chain, we need to derivate to CB58 format. Then, we can import these encoded versions of the private key to the node.

That's over for the `setup`. We are ready to listen to the smart contract to emit the "Staked" event.

#### Adding the event listener part here

Now, we will start the C --&gt; P cross-chain transfer. First, make sure to have some funds on your C-chain address. Go to the "AVAX test faucet": [https://faucet.avax-test.network/](https://faucet.avax-test.network/). Copy and paste your C-chain address from the Avalanche wallet and request some AVAX.

```text
xChainAddress = xKeyChain.getAddressStrings();
cChainAddress = cKeyChain.getAddressStrings();
pChainAddress = pKeyChain.getAddressStrings();
    
let amount = new avalanche.BN("1000000000"); //1 AVAX
    
let cChainHexAddress = "paste your hex address here";
    
let nonce = await web3.eth.getTransactionCount(cChainHexAddress, "pending");
    
let unsignedCtoXTx = await cChain.buildExportTx(
    amount,
    cAvaxAssetId,
    xChainBlockchainID,
    cChainHexAddress,
    cChainAddress[0],
    xChainAddress,
    nonce,
)
    
let signedCtoXTx = await unsignedCtoXTx.sign(cKeyChain);
    
let CtoXTxId = await cChain.issueTx(signedCtoXTx);
    
console.log("CtoXTxId " + CtoXTxId);
    
let fee = cChain.getDefaultTxFee();
    
amount = amount.sub(fee);
    
return new Promise(function (resolve, reject) {
    pollTransaction(waitForStatusC, CtoXTxId, resolve, reject);
}).then(async (resolve) => {
    if (resolve === "Accepted") {
        utxoset = (await xChain.getUTXOs(
            xChainAddress,
            cChainBlockchainID,
        )).utxos;
            
        let unsignedImportXTx = await xChain.buildImportTx(
            utxoset,
            xChainAddress,
            cChainBlockchainID,
            xChainAddress,
            xChainAddress,
        )
            
        let signedImportXTx = await unsignedImportXTx.sign(xKeyChain);
            
        let importXTx = await xChain.issueTx(signedImportXTx);
            
        console.log("importXTx " + importXTx);
            
        fee = xChain.getDefaultTxFee();
            
        amount = amount.sub(fee);
            
        // C --> X done, now let's start X --> P.
            
        utxoset = (await xChain.getUTXOs(
            xChainAddress
        )).utxos;
            
        let unsignedXtoPTx = await xChain.buildExportTx(
            utxoset,
            amount,
            pChainBlockchainID,
            pChainAddress,
            xChainAddress,
        )
            
        let signedXtoPTx = unsignedXtoPTx.sign(xKeyChain);
            
        let XtoPTxId = await xChain.issueTx(signedXtoPTx);
            
        console.log("XtoPTxId " + XtoPTxId);
            
        fee = xChain.getDefaultTxFee();
            
        amount = amount.sub(fee);
            
        return new Promise(function (resolve, reject) {
            pollTransaction(waitForStatusX, XtoPTxId, resolve, reject);
        }).then(async (resolve) => {
            if (resolve === "Accepted") { // ... And import the transaction on the P chain.
                utxoset = (await pChain.getUTXOs(
                    pChainAddress,
                    pChainBlockchainID
                )).utxos;
                
                let unsignedImportPTx = await pChain.buildImportTx(
                    utxoset,
                    pChainAddress,
                    xChainBlockchainID,
                    pChainAddress,
                    pChainAddress,
                )
                    
                let signedImportPTx = unsignedImportPTx.sign(pKeyChain);
                    
                let importPTx = await pChain.issueTx(signedImportPTx);
                    
                console.log("importPTx " + importPTx);
                    
                fee = pChain.getDefaultTxFee();
            
                amount = amount.sub(fee);
                    
                await stakeToNode(nodeId)
            } else {
                console.log("An error happened for a transaction with ID: " + XtoPTxId)
            }
        })
    } else {
        console.log("An error happened for a transaction with ID: " + CtoXTxId)
    }
})
```

This code snippet is doing a cross-chain transfer between the C-chain and P-chain with 1 $AVAX, deducting the fees for each transaction.

We cannot directly do a C-chain to P-chain transaction, but we can do C-chain to X-chain and X-chain to P-chain. For each transaction export, we need to wait for the transaction to be marked as "Accepted" to import the transaction.

We are now ready to delegate to a node now that the funds are on the P-chain.

