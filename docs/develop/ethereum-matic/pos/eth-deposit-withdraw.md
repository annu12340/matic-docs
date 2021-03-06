---
id: eth-deposit-withdraw
title: ETH Deposit and Withdraw Guide
description: Build your next blockchain app on Matic.
keywords:
  - docs
  - matic
image: https://matic.network/banners/matic-network-16x9.png 
---
## High Level Flow

Deposit ETH -

1. Make ***depositEtherFor*** call on ***RootChainManager*** and ******send ******the ******required ether. 

Withdraw ETH -

1. ***Burn*** tokens on matic chain.
2. Call ***exit*** function on ***RootChainManager*** to submit proof of burn transaction. This call can be made ***after checkpoint*** is submitted for the block containing burn transaction.

## Step Details
---

### Configuring Matic SDK

Install Matic SDK (***2.0.2)***

```bash
npm install --save @maticnetwork/maticjs
```

```json
"dependencies": {
    "@maticnetwork/maticjs": "2.0.2"
}
```

While creating ***MaticPOSClient*** object ***maticProvider***, ***parentProvider***, ***rootChain*** and ***posRootChainManager*** need to be provided.

```jsx
const MaticPOSClient = require('@maticnetwork/maticjs').MaticPOSClient

const maticPOSClient = new MaticPOSClient({
    network: 'testnet', // optional, default is testnet
    version: 'mumbai', // optional, default is mumbai
    parentProvider: new HDWalletProvider(config.user.privateKey, config.root.RPC),
    maticProvider: new HDWalletProvider(config.user.privateKey, config.child.RPC),
    posRootChainManager: config.root.POSRootChainManager,
    posERC20Predicate: config.root.posERC20Predicate, // optional, required only if working with ERC20 tokens
    posERC721Predicate: config.root.posERC721Predicate, // optional, required only if working with ERC721 tokens
    posERC1155Predicate: config.root.posERC1155Predicate, // optional, required only if working with ERC71155 tokens
    parentDefaultOptions: { from: config.user.address }, // optional, can also be sent as last param while sending tx
    maticDefaultOptions: { from: config.user.address }, // optional, can also be sent as last param while sending tx
})
```
### Deposit

ETH can be deposited to matic chain by calling ***depositEtherFor*** on RootChainManager contract. Matic POS client exposes ***depositEtherForUser*** method to make this call.

***ETH*** is deposited as ***ERC20*** token on Matic chain. For withdrawing it follow the same process as withdrawing ERC20 tokens.

```jsx
await maticPOSClient.depositEtherForUser(from, amount, { from, gasPrice: '10000000000' })
```

### Burn

User can call ***withdraw*** function of ***ChildToken*** contract. This function should burn the tokens. Matic POS client exposes ***burneth*** method to make this call.

```jsx
await maticPOSClient.burneth(childToken, amount, { from })
```

Store the transaction hash for this call and use it while generating burn proof.

### Exit

Once the ***checkpoint*** has been ***submitted*** for the block containing burn transaction, user should call the ***exit*** function of ***RootChainManager*** contract and submit the proof of burn. Upon submitting valid proof tokens are transferred to the user. Matic POS client exposes ***exiteth*** method to make this call.

```jsx
await maticPOSClient.exiteth(burnTxHash, { from })
```