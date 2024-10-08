# A51 Finance API Documentation

## Introduction

The **A51 Finance** backend provides various APIs for handling interactions with its decentralized finance platform. This documentation outlines the primary endpoints, authentication mechanisms, and the expected data formats for interacting with A51 Finance DApp. The frontend is built with **Material Design** using **@mui v5**, and the backend is structured to handle user requests through authenticated API calls.

## Server API URIs

- **Swagger Playground**: [Swagger API](https://api-dev.a51.finance/api/)
- **Production**: [Production API](https://api.a51.finance)
- **Development**: [Development API](https://api-dev.a51.finance)

## Authentication

For any API that modifies data (e.g., `POST`, `PUT`), an authentication token is required. The authentication process involves signing a message through the user's wallet. This signed message is then used as the authentication token in the request header.

### Signing Message Example

The user signs the following message using their wallet (e.g., Metamask):

```
SIGN_MESSAGE = "Greetings from A51!

To authenticate, kindly use the 'Sign' button.

It's important to note that this action won't initiate any blockchain transactions or incur gas fees."
```

This signed message is then passed in the authentication header for further API calls.

## Strategy Creation

To create a new strategy, the frontend integrates the `CLTBase` contract using the `createStrategy` function.

### API Endpoint

```
POST {serverBaseUrl}/store/pre-tx-strategy
```

### Prerequisites

- A valid pool must exist and be initialized on the relevant DEX and blockchain.
- Pool information can be fetched using the `pool-stats` API.

Example:

```
GET {serverBaseUrl}/subgraphs/pool-stats?token0=0x...&token1=0x...&protocol=baseswap-uniswap-v3-cloned&chainId=8453
```

### Contract Function

```solidity
function createStrategy(
    StrategyKey calldata key,
    PositionActions calldata actions,
    uint256 managementFee,
    uint256 performanceFee,
    bool isCompound,
    bool isPrivate
);
```

#### Parameters

- `key`: Contains `poolAddress`, `minPriceTick`, and `maxPriceTick`.
- `managementFee`: Fee charged for managing the strategy.
- `performanceFee`: Fee based on the strategy's performance.
- `isCompound`: Boolean to enable or disable compounding.
- `isPrivate`: Boolean to determine if the strategy is private.
- `actions`: Contains various actions like `mode`, `exitStrategy`, `liquidityDistribution`, etc.

### Example Request

```json
{
  "pool": "0x...",
  "tickLower": 12000,
  "tickUpper": 15000,
  "managementFee": 2000000000000000000,
  "performanceFee": 1000000000000000000,
  "isCompound": true,
  "isPrivate": false,
  "actions": {
    "mode": "DYNAMIC",
    "exitStrategy": [],
    "liquidityDistribution": [],
    "rebaseStrategy": [
      {
        "actionName": "0x697d...",
        "data": "0x..."
      }
    ]
  }
}
```

### Response

Upon success, a strategy ID is returned, which can be used to interact with the strategy.

## Strategy Overview and Actions

Users can view all strategies through the following API endpoint:

```
GET {serverUrl}/subgraphs/a51-strategies
```

### Strategy Details

For each strategy, users can view:

- **Owner Information**
- **Pool Address**
- **Management and Performance Fees**
- **Liquidity Distribution**
- **Compounding Status**
- **Position Balances**

### Example Strategy Response

```json
{
  "strategyName": "My Strategy",
  "owner": "0x123...",
  "managementFee": 0.02,
  "performanceFee": 0.1,
  "isCompound": true,
  "balance0": 500,
  "balance1": 300,
  "mode": "DYNAMIC"
}
```

## Deposit and Liquidity Calculation

To compute the deposit ratio when the strategy has no assets under management (AUM), the backend calculates token reserves using the `CLTBase` and `CLTHelper` contracts.

## Deposit

We have two methods for computing the deposit ratio when we have 0 AUM (Assets Under Management) in the strategy:

1. **Uniswap Calculation**: If there is no AUM, the deposit value will be calculated the same way Uniswap calculates it, based on ticks.
2. **Custom Calculation**: If AUM exists, we will use our own ratio calculation based on **TokenReserve0** and **TokenReserve1** since some users' assets might be in the contract but not utilized in the liquidity.

## Single-Asset Deposit

The Single-Asset Deposit feature allows users to deposit liquidity into a strategy even if they hold only one of the two pool tokens. This feature streamlines the liquidity provision process by enabling the deposit of a single asset, which will be automatically swapped into the other token in the pool.

## How to Use the Single-Asset Deposit Feature

### 1. Select a Strategy

Navigate to the strategy page, where three deposit options are available: **"Dual Asset," "Single Asset,"** and **"Zappin."**

### 2. Choose the Single-Asset Option

Select the **"Single Asset"** tab to proceed with the deposit. A dropdown will appear, allowing the selection of one of the two pool tokens for deposit.

### 3. Token Approval

Before depositing, approval for the selected token must be granted. This step authorizes the platform to interact with the token on behalf of the user.

### 4. Deposit Process

After the token is approved, click the **"Deposit"** button. The selected single asset will automatically be swapped into the other token of the pool, based on the current pool ratio. Once the swap is completed, liquidity will be deposited into the pool.

### 5. View Liquidity Position

Once the deposit process is finalized, the liquidity position will be visible, showing holdings in both tokens of the pool.

---

This feature simplifies liquidity provision by allowing deposits with just one asset, with automatic rebalancing into the appropriate pool ratio.

## Zappin

The Zappin feature enables users to deposit liquidity into a strategy using any asset token, even if it is not one of the pool tokens. This feature provides flexibility by allowing deposits with a wide range of tokens, which are automatically swapped into the appropriate pool tokens based on the pool's ratios.

## How to Use the Zappin Feature

### 1. Select a Strategy

Navigate to the strategy page, where three deposit options are available: **"Dual Asset," "Single Asset,"** and **"Zappin."**

### 2. Choose the Zappin Option

Select the **"Zappin"** tab to proceed with the deposit. A dropdown will appear, displaying various tokens that can be used for the deposit, including tokens outside of the pool pair.

### 3. Token Approval

Before proceeding with the deposit, approve the selected token. This authorization allows the platform to interact with the token on behalf of the user.

### 4. Deposit Process

After token approval, click the **"Deposit"** button. The selected asset will automatically be swapped into the two pool tokens based on the current pool ratios. The swap ensures that the correct proportion of each pool token is deposited.

### 5. View Liquidity Position

Once the deposit is completed, the liquidity position will reflect holdings in both tokens of the pool, aligned with the ongoing pool ratio.

---

The Zappin feature adds flexibility by allowing users to deposit with various tokens, making it easier to provide liquidity even if the user doesn't hold the specific pool tokens. The platform handles the necessary conversions and rebalancing to ensure proper liquidity provision.

### Contract Functions

We utilize **CLTBase** and **CLTHelper** contract functions for the following operations:

### Uncompounded Fee Calculation

- First, we call the `CLTBase` contract's `getStrategyReserves` function to retrieve **uncompoundedFee0** and **uncompoundedFee1**.

### Strategy Information

- We gather important strategy information such as:
  - **balance0** and **balance1**
  - **liquidity**
  - **poolAddress**
  - **tickLower** and **tickUpper**
  - **isCompound**

This data is collected by calling the `strategies` function in the **CLTBase** contract.

### Liquidity Reserve Calculation

- Once we have the tick and liquidity data, we pass **tickLower**, **tickUpper**, and **liquidity** to get the liquidity reserve from the DEX (Decentralized Exchange) in terms of **token0** and **token1** using the **CLTHelper** function `getStrategyReserves`.

### Compounded Strategy Handling

- If the strategy is compounded, we include the **uncompoundedFee**, **balance**, and add the **DEX reserve**.
- If the strategy is **not compounded**, we will exclude the **uncompoundedFee**.

Here is an example for compounded and non-compounded strategies:

```javascript
if (isCompound) {
    return {
        poolAddress,
        tokenReserves0: balance0.add(uncompoundedFee0).add(dexReserves0),
        tokenReserves1: balance1.add(uncompoundedFee1).add(dexReserves1),
        tickLower: Number(tickLower),
        tickUpper: Number(tickUpper),
    };
}

return {
    poolAddress,
    tokenReserves0: balance0.add(dexReserves0),
    tokenReserves1: balance1.add(dexReserves1),
    tickLower: Number(tickLower),
    tickUpper: Number(tickUpper),
};


## Strategy Management

Strategy owners can manage and update their strategy details through the following API:

```

PUT {serverUrl}/store/strategy/{chainId}/{protocol}

````

### Payload
```json
{
  "id": "strategyId",
  "name": "Updated Strategy",
  "desc": "An updated description for my strategy",
  "twitter": {
    "name": "John Doe",
    "photoUrl": "https://example.com/johndoe.jpg",
    "username": "john_doe"
  }
}
````

### Liquidity Calculation Example

```javascript
const ratio = token0.equals(tokenA)
  ? token1Reserves.dividedBy(token0Reserves)
  : token0Reserves.dividedBy(token1Reserves);

const dependentValue = ratio.multipliedBy(typedValue).toString();
```

## API Responses

Each API response contains the following standard fields where applicable:

- **Strategy Name**: The name of the strategy.
- **Strategy Description**: A detailed description of the strategy.
- **Fees**: Management and performance fees.
- **Balances**: Token balances associated with the strategy.
- **Total Earned Tokens**: Cumulative earnings in terms of tokens.
- **Annual Percentage Rate (APR)**: Weekly or annualized APR if available.

## Manual Readjust

In the **Automation** tab, the strategy owner can update the strategy's information, including:

- The liquidity price range (**lowerTick** and **upperTick**)
- Whether to hold users' liquidity or not
- The swap amount between **TokenA** and **TokenB** (either in the **zeroForOne** direction or the reverse)

### Contract Interface

You can refer to the **rebaseModule** contract interface below:

```solidity
type ExectuteStrategyParamsStruct = {
    pool: string;
    strategyID: BytesLike;
    tickLower: BigNumberish;
    tickUpper: BigNumberish;
    shouldMint: boolean;
    zeroForOne: boolean;
    swapAmount: BigNumberish;
    sqrtPriceLimitX96: BigNumberish;
    isRebaseToken: boolean;
};

function executeStrategy(
    executeParams: IRebaseStrategy.ExectuteStrategyParamsStruct
);

```

## Strategy Parameters

- **poolAddress**: The address of the pool.
- **tickLower**: The lower bound of the liquidity price range.
- **tickUpper**: The upper bound of the liquidity price range.
- **strategyID**: The unique identifier for the strategy.

### Rebase Token

- **isRebaseToken**: Currently set to `false`.

### Liquidity Management

- **shouldMint**: Determines whether to limit liquidity on Baseswap or hold it.
  - If the toggle in the UI is **true**, pass **false** (e.g., `!holdLiquidity`).

### Slippage and Price Calculation

- **SqrtPriceLimitX96**: Takes slippage into account and considers the direction of the swap (either **zero-for-one** or **one-for-zero**).

#### Fetch `sqrtPrice`

- This value is retrieved from the pool contract.

### Calculate `sqrtPriceLimitX96`

To calculate the `sqrtPriceLimitX96`, we consider the direction of the swap (**zero-for-one** or **one-for-zero**) and adjust based on the slippage percentage.

```javascript
const sqrtPriceLimitX96 = zeroForOneTokenSwap
  ? sqrtPriceX96
      .minus(sqrtPriceX96.multipliedBy(swapSlippagePercentage / 100))
      .toString()
  : sqrtPriceX96
      .plus(sqrtPriceX96.multipliedBy(swapSlippagePercentage / 100))
      .toString();
```

- If zeroForOneTokenSwap is true, subtract the slippage percentage from sqrtPriceX96.
- If zeroForOneTokenSwap is false, add the slippage percentage to sqrtPriceX96

## Conclusion

The A51 Finance backend APIs provide comprehensive support for strategy management, liquidity calculation, and user interactions with blockchain-based assets. Users can interact with strategies, manage their assets, and track performance using the endpoints provided above. Ensure that all mutating API requests include a signed authentication token for secure transactions.

For further information, visit the **[Swagger API Documentation](https://api-dev.a51.finance/api/)**.
