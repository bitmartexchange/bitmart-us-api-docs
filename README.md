# BitMart US OpenAPI Documentation

## Overview

This document describes the BitMart US OpenAPI interface specifications for spot trading, account management, and market data queries.

**Prod Environment Base URL:** `https://api-cloud.bitmart.us`

**API Version:** v1

---

## Authentication

All private endpoints require HMAC-SHA256 signature authentication.

### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| X-BM-KEY | Yes | API Access Key |
| X-BM-TIMESTAMP | Yes | Request timestamp (milliseconds) |
| X-BM-SIGN | Yes | HMAC-SHA256 signature |
| Content-Type | Yes (POST) | `application/json` |

### Signature Algorithm

```
signature = HMAC-SHA256(secretKey, timestamp + "#" + memo + "#" + body)
```

- **GET requests:** `body` is the JSON format of query parameters, e.g., `{"symbol":"BTC/USD"}`
- **POST requests:** `body` is the JSON request body

### Example

```bash
curl -X GET 'https://api-cloud-us.bmexchangeaws-test.com/bm-us/api/v1/account' \
  -H 'X-BM-KEY: your_access_key' \
  -H 'X-BM-TIMESTAMP: 1702540800000' \
  -H 'X-BM-SIGN: your_signature'
```

---

## Response Format

All API responses follow a unified format:

```json
{
  "code": 0,
  "message": "success",
  "data": { ... }
}
```

| Field | Type | Description |
|-------|------|-------------|
| code | Integer | Response code, `0` indicates success |
| message | String | Response message |
| data | Object | Response data |

---

## Market Data APIs (Public)

### Get Server Time

Get the server timestamp for time synchronization.

**Endpoint:** `GET /api/v1/time`

**Authentication:** Not required

**Request Parameters:** None

**Response Example:**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "server_time": 1702540800000
  }
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| server_time | Long | Server timestamp (milliseconds) |

---

### Get Exchange Info

Get trading pair information including trading rules and precision limits.

**Endpoint:** `GET /api/v1/exchange-info`

**Authentication:** Not required

**Request Parameters:** None

**Response Example:**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "messages": [
      {
        "product_id": "BTC",
        "symbol": "BTC/USD",
        "description": "Bitcoin",
        "image_url": "https://...",
        "fractional_qty_scale": "100000000",
        "price_scale": "100",
        "tick_size": "0.01",
        "last_price": "86014.63",
        "change_percent": "0",
        "timestamp": "1702540800000",
        "minimum_trade_qty": "1",
        "price_limit": {
          "relative_low": 0.6,
          "relative_high": 0.6
        },
        "order_size_limit": {
          "total_notional_low": "10000000000",
          "total_notional_high": "5000000000000000"
        }
      }
    ]
  }
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| messages | Array | List of trading pairs |
| messages[].product_id | String | Product ID |
| messages[].symbol | String | Trading pair symbol, e.g., BTC/USD |
| messages[].description | String | Trading pair description |
| messages[].image_url | String | Coin icon URL |
| messages[].fractional_qty_scale | String | Quantity precision multiplier (e.g., 100000000 = 8 decimals) |
| messages[].price_scale | String | Price precision multiplier (e.g., 100 = 2 decimals) |
| messages[].tick_size | String | Minimum price increment |
| messages[].last_price | String | Latest trade price |
| messages[].change_percent | String | 24-hour price change percentage |
| messages[].timestamp | String | Update timestamp (milliseconds) |
| messages[].minimum_trade_qty | String | Minimum trade quantity (scaled) |
| messages[].price_limit | Object | Price limit configuration |
| messages[].price_limit.relative_low | Number | Min price ratio (min_price = last_price × relative_low) |
| messages[].price_limit.relative_high | Number | Max price ratio (max_price = last_price × (1 + relative_high)) |
| messages[].order_size_limit | Object | Order amount limit configuration |
| messages[].order_size_limit.total_notional_low | String | Minimum order amount (scaled) |
| messages[].order_size_limit.total_notional_high | String | Maximum order amount (scaled) |

**Calculate actual limits:**
- Min order amount: `total_notional_low / fractional_qty_scale / price_scale`
- Max order amount: `total_notional_high / fractional_qty_scale / price_scale`

---

### Get Ticker Price

Get the latest price for a symbol or all symbols.

**Endpoint:** `GET /api/v1/ticker/price`

**Authentication:** Not required

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | String | No | Trading pair (e.g., BTC/USD). If empty, returns all symbols |

**Response Example:**

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "symbol": "BTC/USD",
      "price": "86014.63",
      "change_percent": "1.25",
      "timestamp": "1702540800000"
    }
  ]
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| symbol | String | Trading pair symbol |
| price | String | Latest price |
| change_percent | String | 24-hour price change percentage |
| timestamp | String | Timestamp (milliseconds) |

---

## Account APIs (Private)

### Get Account Info

Get all asset balances.

**Endpoint:** `GET /api/v1/account`

**Authentication:** Required

**Request Parameters:** None

**Response Example:**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "balances": [
      {
        "asset": "BTC",
        "free": "1.5",
        "locked": "0.5",
        "total": "2.0"
      }
    ]
  }
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| balances | Array | Balance list |
| balances[].asset | String | Asset name |
| balances[].free | String | Available balance |
| balances[].locked | String | Locked balance |
| balances[].total | String | Total balance |

---

### Get Balance

Get balance for a specific asset.

**Endpoint:** `GET /api/v1/balance`

**Authentication:** Required

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| asset | String | Yes | Asset name (e.g., BTC) |

**Response Example:**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "asset": "BTC",
    "free": "1.5",
    "locked": "0.5",
    "total": "2.0"
  }
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| asset | String | Asset name |
| free | String | Available balance |
| locked | String | Locked balance |
| total | String | Total balance |

---

### Get Deposit History

Get deposit history records.

**Endpoint:** `GET /api/v1/deposit/history`

**Authentication:** Required

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| coin | String | No | Coin name |
| startTime | Long | No | Start timestamp (ms) |
| endTime | Long | No | End timestamp (ms) |
| limit | Integer | No | Limit (default: 500, max: 1000) |

**Response Example:**

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "tx_id": "abc123def456...",
      "asset": "BTC",
      "amount": "0.1",
      "address": "bc1q...",
      "status": "SUCCESS",
      "create_time": 1702540800000,
      "complete_time": 1702541000000
    }
  ]
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| tx_id | String | Transaction hash |
| asset | String | Asset name |
| amount | String | Deposit amount |
| address | String | Deposit address |
| status | String | Status: PENDING, SUCCESS, FAILED |
| create_time | Long | Creation time (milliseconds) |
| complete_time | Long | Completion time (milliseconds) |

---

### Get Withdraw History

Get withdrawal history records.

**Endpoint:** `GET /api/v1/withdraw/history`

**Authentication:** Required

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| coin | String | No | Coin name |
| startTime | Long | No | Start timestamp (ms) |
| endTime | Long | No | End timestamp (ms) |
| limit | Integer | No | Limit (default: 500, max: 1000) |

**Response Example:**

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "tx_id": "abc123def456...",
      "asset": "BTC",
      "amount": "0.1",
      "received_amount": "0.0999",
      "fee": "0.0001",
      "address": "bc1q...",
      "status": "COMPLETED",
      "create_time": 1702540800000,
      "complete_time": 1702541000000
    }
  ]
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| tx_id | String | Transaction hash |
| asset | String | Asset name |
| amount | String | Withdrawal amount |
| received_amount | String | Actual received amount |
| fee | String | Transaction fee |
| address | String | Withdrawal address |
| status | String | Status: PENDING, PROCESSING, COMPLETED, FAILED, CANCELED |
| create_time | Long | Creation time (milliseconds) |
| complete_time | Long | Completion time (milliseconds) |

---

### Get Trade History

Get trade execution history.

**Endpoint:** `GET /api/v1/trades`

**Authentication:** Required

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| symbol | String | Yes | Trading pair (e.g., BTC/USD) |
| startTime | Long | No | Start timestamp (ms) |
| endTime | Long | No | End timestamp (ms) |
| limit | Integer | No | Limit (default: 500, max: 1000) |

**Response Example:**

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "trade_id": "123456",
      "order_id": "789012",
      "symbol": "BTC/USD",
      "price": "86000.00",
      "qty": "0.001",
      "quote_qty": "86.00",
      "commission": "0.068",
      "time": 1702540800000,
      "is_buyer": true
    }
  ]
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| trade_id | String | Trade ID |
| order_id | String | Order ID |
| symbol | String | Trading pair |
| price | String | Trade price |
| qty | String | Trade quantity |
| quote_qty | String | Trade amount |
| commission | String | Commission fee |
| time | Long | Trade time (milliseconds) |
| is_buyer | Boolean | Whether the trade was a buy, true=buy, false=sell |

---

## Address APIs (Private)

### Get Address List

Get deposit addresses.

**Endpoint:** `GET /api/v1/address`

**Authentication:** Required

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| coin | String | No | Coin name. If empty, returns all coins |

**Response Example:**

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "asset": "BTC",
      "address": "bc1q..."
    }
  ]
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| asset | String | Asset name |
| address | String | Deposit address |

---

### Create Address

Create a new deposit address.

**Endpoint:** `POST /api/v1/address`

**Authentication:** Required

**Request Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| coin | String | Yes | Coin name |

**Request Example:**

```json
{
  "coin": "BTC"
}
```

**Response Example:**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "asset": "BTC",
    "address": "bc1q..."
  }
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| asset | String | Asset name |
| address | String | Newly created deposit address |

---

## Order APIs (Private)

### Create Order

Create a new spot order.

**Endpoint:** `POST /api/v1/orders`

**Authentication:** Required

**Request Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| symbol | String | Yes | Trading pair (e.g., BTC/USD) |
| side | String | Yes | Order side: `BUY` or `SELL` |
| type | String | Yes | Order type: `LIMIT` or `MARKET` |
| price | String | Conditional | Price (required for LIMIT orders), scaled value |
| quantity | String | Conditional | Quantity (scaled value), required for LIMIT or MARKET by qty |
| cash_order_qty | String | Conditional | Amount (scaled value), for MARKET orders by amount |
| time_in_force | String | No | Time in force (default: `TIME_IN_FORCE_GOOD_TILL_CANCEL`) |
| client_order_id | String | No | Custom client order ID |

**Time In Force Options:**

| Value | Description |
|-------|-------------|
| TIME_IN_FORCE_GOOD_TILL_CANCEL | Good till cancelled |
| TIME_IN_FORCE_DAY | Valid for the day |
| TIME_IN_FORCE_IMMEDIATE_OR_CANCEL | Immediate or cancel |
| TIME_IN_FORCE_FILL_OR_KILL | Fill or kill |

**Precision Calculation:**

All price and quantity values must be multiplied by the corresponding scale before submission:

```
scaled_price = price × price_scale
scaled_quantity = quantity × fractional_qty_scale
scaled_cash_amount = amount × fractional_qty_scale × price_scale
```

**Price Limit Validation (LIMIT orders only):**

```
min_price = last_price × relative_low
max_price = last_price × (1 + relative_high)
```

**Order Amount Validation:**

```
min_amount = total_notional_low / fractional_qty_scale / price_scale
max_amount = total_notional_high / fractional_qty_scale / price_scale
```

**Request Example - Limit Buy Order:**

```json
{
  "symbol": "BTC/USD",
  "side": "BUY",
  "type": "LIMIT",
  "price": "8171390",
  "quantity": "10000",
  "time_in_force": "TIME_IN_FORCE_GOOD_TILL_CANCEL"
}
```

**Request Example - Market Buy Order (by amount):**

```json
{
  "symbol": "BTC/USD",
  "side": "BUY",
  "type": "MARKET",
  "cash_order_qty": "100000000000",
  "time_in_force": "TIME_IN_FORCE_GOOD_TILL_CANCEL"
}
```

**Response Example:**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "order_id": "6WW511S0A021"
  }
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| order_id | String | Order ID |

---

### Cancel Order

Cancel an existing order.

**Endpoint:** `POST /api/v1/orders/cancel`

**Authentication:** Required

**Request Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| order_id | String | Yes | Order ID |

**Request Example:**

```json
{
  "order_id": "6WW511S0A021"
}
```

**Response Example:**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "result": true
  }
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| result | Boolean | Cancel result, true=success, false=failed |

---

### Query Order

Get order details by order ID.

**Endpoint:** `GET /api/v1/orders`

**Authentication:** Required

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| order_id | String | Yes | Order ID |

**Response Example:**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "order_id": "6WW511S0A021",
    "symbol": "BTC/USD",
    "side": "BUY",
    "type": "LIMIT",
    "price": "8171390",
    "order_qty": "10000",
    "filled_qty": "5000",
    "avg_price": "8171000",
    "status": "PARTIALLY_FILLED",
    "create_time": "2024-01-15T10:30:00.000Z"
  }
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| order_id | String | Order ID |
| symbol | String | Trading pair |
| side | String | Order side: BUY=buy, SELL=sell |
| type | String | Order type: LIMIT=limit order, MARKET=market order |
| price | String | Order price (scaled) |
| order_qty | String | Order quantity (scaled) |
| filled_qty | String | Filled quantity (scaled) |
| avg_price | String | Average fill price (scaled) |
| status | String | Order status |
| create_time | String | Creation time (ISO 8601 format) |

**Order Status:**

| Status | Description |
|--------|-------------|
| NEW | New order |
| PARTIALLY_FILLED | Partially filled |
| FILLED | Fully filled |
| CANCELED | Cancelled |
| REJECTED | Rejected |

---

## Error Codes

| Code | Description |
|------|-------------|
| 0 | Success |
| 1001 | Parameter error |
| 1002 | User not found |
| 2001 | Insufficient balance |
| 2002 | Order not found |
| 2003 | Price out of range |
| 2004 | Order amount out of range |
| 3001 | System error |

---

## Rate Limits

- **Public endpoints:** 10 requests/second per IP
- **Private endpoints:** 20 requests/second per API key
- **Order endpoints:** 5 requests/second per API key

---

## Change Log

| Version | Date       | Changes |
|---------|------------|---------|
| v1.0.0 | 2025-12-19 | Initial release |
