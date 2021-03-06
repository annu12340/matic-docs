---
id: erc1155-deposit-withdraw
title: ERC1155 Deposit and Withdraw Guide
description: Build your next blockchain app on Matic.
keywords:
  - docs
  - matic
image: https://matic.network/banners/matic-network-16x9.png 
---
## High Level Flow

Deposit ERC1155 -

1. ***Approve*** ***RootChainManager*** contract to spend the tokens that have to be deposited.
2. Make ***depositFor*** call on ***RootChainManager***.

Withdraw ERC1155 -

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

### Approve

This is a normal ERC1155 approval so that  ***RootChainManager*** can call ***transferFrom*** function. Matic POS client exposes ***approveERC1155ForDeposit*** method to make this call.

```jsx
await maticPOSClient.approveERC1155ForDeposit(rootToken, tokenId, { from })
```

### Deposit

Deposit can be done by calling ***depositFor*** on RootChainManager contract. Note that token needs to be mapped and approved for transfer beforehand. Once tokens are transferred deposit proceeds using StateSync mechanism. Matic POS client exposes ***depositSingleERC1155ForUser*** & ***depositBatchERC1155ForUser*** method to make this call.

```jsx
await maticPOSClient.depositSingleERC1155ForUser(rootToken, from, tokenId, amount, { from, gasPrice: '10000000000' })
```

```jsx
await maticPOSClient.depositBatchERC1155ForUser(rootToken, from, [...tokenId], [...amount], { from, gasPrice: '10000000000' })
```

***deposit*** function of ***ChildToken*** is called by the ***ChildChainManager.*** Tokens should be minted when this call is made.

### Burn

User can call ***withdraw*** function of ***ChildToken*** contract. This function should burn the tokens. Matic POS client exposes ***burnSingleERC1155*** & ***burnBatchERC1155*** method to make this call.

```jsx
await maticPOSClient.burnSingleERC1155(rootToken, tokenId, amount, { from, gasPrice: '10000000000' })
```

```jsx
await maticPOSClient.burnBatchERC1155(rootToken, [...tokenId], [...amount], { from, gasPrice: '10000000000' })
```

**Store the transaction hash for this call and use it while generating burn proof.**

### Exit

Once the ***checkpoint*** has been ***submitted*** for the block containing burn transaction, user should call the ***exit*** function of ***RootChainManager*** contract and submit the proof of burn. Upon submitting valid proof tokens are transferred to the user. Matic POS client exposes ***exitBatchERC1155*** & ***singleBatchERC1155***  method to make this call.

```jsx
await maticPOSClient.exitSingleERC1155(burnTxHash, { from })
```

```jsx
await maticPOSClient.exitBatchERC1155(burnTxHash, { from })
```