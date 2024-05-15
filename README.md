# Immutable Support Queries

## Table of Contents

1. 500 Internal Server Error on API Call
2. Smart Contract Deployment Error on Immutable zkEVM

## #1 - 500 Internal Server Error on API Call

### Issue

User reported encountering a 500 Internal Server Error when calling `https://api.sandbox.immutable.com/v1/transfers`. The error reoccured without any changes to their script.

### Possible Reasons for Error

1. **Wide Timeframe**: The query might be covering a wide timeframe, leading to the request to timeout before the response is received
2. **Unindexed Filters**: The endpoint might be using an unindexed filter causing longer query times resulting in timeouts

### Suggested Approach

To address the 500 Internal Server Error, you can consider the following:

**Using Filters & Reduce Query Volume**:
   - Use the `order_by` fields (`desc` / `asc`) to sort the data
   - Reduce `page_size` (Query returns 100 values by default)
   - `user` (Filter for a specific user who executed the transfer)
   - `token_address` (Filter for a specific token address)
   - `token_type` (ERC20 / ERC721)
   - `receiver_address = 0` (Tracking for Burn Events specifically)
   
### Sample Query with Filters

```javascript
import axios from 'axios';

const options = {
  method: 'GET',
  url: 'https://api.sandbox.x.immutable.com/v1/transfers',
  params: {
    order_by: 'transaction_id',
    direction: 'desc',
    receiver: '0x0000000000000000000000000000000000000000',
    status: 'success',
    min_quantity: '1'
  },
  headers: { Accept: 'application/json' }
};

try {
  const { data } = await axios.request(options);
  console.log(data);
} catch (error) {
  console.error(error);
}
```

### Next Steps

If the issue still persists after applying the suggested filters, kindly provide us with the following information: 

- A detailed description of the issue
- The exact API request being made (including endpoint, parameters, filters used)
- Any relevant error logs or screenshots

This information will greatly assist us in resolving your issue.

## #2 - Smart Contract Deployment Error on Immutable zkEVM

### Issue

User reported an error when deploying a smart contract on Immutable zkEVM with an internal JSON-RPC error with a error message indicating that the transaction is underpriced.

### Reasons for Error

Transactions on Immutable zkEVM requires a **minimum gas price of 10 gwei** to protect the network against spam traffic.

### Explaination

Transactions with a tip cap below 10 gwei are by default rejected by the RPC. The fee cap must be greater than or equal to 10 gwei in order for the transaction to be executed.

## Solution

To fix this issue, refer to the sample `deploy.ts` script below on how to price the smart contract deployment transaction by adding the `gasOverrides` parameter:

```typescript
import { ethers } from "hardhat";
import * as dotenv from "dotenv";

dotenv.config();

async function main() {
  // Load the contract and get the contract factory
  const MyContract = await ethers.getContractFactory("MyContract");

  const [deployer] = await ethers.getSigners();
  console.log("Deploying Contract with:", deployer.address);

  const owner = deployer.address;
  const name = "My First Contract";
  const symbol = "POC";
  const baseURI = "https://example.com/metadata/";
  const contractURI = "https://example.com/contract-metadata/";
  const receiver = deployer.address;
  const operatorAllowlist = "0x6b969fd89de634d8de3271ebe97734feffcd58ee";
  const feeNumerator = 1000; // Example value, change as needed

  const estimatedGasLimit = await ethers.provider.estimateGas(
    await MyContract.getDeployTransaction(
      owner,
      name,
      symbol,
      baseURI,
      contractURI,
      operatorAllowlist,
      receiver,
      feeNumerator
    )
  );

  console.log("Estimated Gas Limit:", estimatedGasLimit.toString());

  const gasOverrides = {
    maxPriorityFeePerGas: 10e9, // 10 gwei
    maxFeePerGas: 15e9, 
    gasLimit: estimatedGasLimit, // Use the estimated gas limit
  };

  // Deploy the contract to the zkEVM network
  const contract = await MyContract.deploy(
    owner,
    name,
    symbol,
    baseURI,
    contractURI,
    operatorAllowlist,
    receiver,
    feeNumerator,
    gasOverrides
  );

  // Wait for the contract to be deployed
  console.log("Contract deployed to:", await contract.getAddress());
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```
This approach should help you resolve the gas pricing issue and successfully deploy your smart contract on Immutable zkEVM!

