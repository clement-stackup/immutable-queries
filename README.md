# Immutable Customer Queries

## Table of Contents

1. 500 Internal Server Error on API Call
2. Smart Contract Deployment Error on Immutable zkEVM

## 500 Internal Server Error on API Call

### Issue

User reported encountering a 500 Internal Server Error when calling `https://api.sandbox.immutable.com/v1/transfers`. The error reoccured without any changes to their script.

### Reasons for Error

1. **Wide Timeframe**: The query might be covering a wide timeframe, leading to the request to timeout before the response is received
2. **Unindexed Filters**: The endpoint might be using an unindexed filter causing longer query times resulting in timeouts

### Suggested Approach

To address the 500 Internal Server Error, you can consider:

1. **Using Filters & Reduce Query Volume**:
   - Use the `order_by` fields (`desc` / `asc`) to sort the data
   - Reduce `page_size` (Query returns 100 values by default)
   - `user` (Filter for a specific user who executed the transfer)
   - `token_address` (Filter for a specific token address - Illuviual Shards)
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

If the issue still persists after applying the suggested filters, do consider the following:

1. Ensure that you are not exceeding the API rate limits and implement retry logic in your script to handle rate limit errors
2. Contact Immutable Support with the following information:

- A detailed description of the issue
- The exact API request being made (Endpoint, Parameters and Headers)
- Any relevant error logs or screenshots
- Time and data of the error

This will help the support team to investigate and provide a more taiilored soluton to your query







