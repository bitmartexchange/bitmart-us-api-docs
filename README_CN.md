# BitMart US OpenAPI 接口文档

## 概述

本文档描述 BitMart US OpenAPI 接口规范，包括现货交易、账户管理、市场数据查询、**充提（虚拟币 / ACH）**及**绑定银行账户**等功能。

**生产环境基础URL：** `https://api-cloud.bitmart.us`

**API版本：** v1

---

## 认证方式

所有私有接口需要 HMAC-SHA256 签名认证。

### 请求头

| 请求头 | 必填 | 描述 |
|--------|------|------|
| X-BM-KEY | 是 | API Access Key |
| X-BM-TIMESTAMP | 是 | 请求时间戳（毫秒） |
| X-BM-SIGN | 是 | HMAC-SHA256 签名 |
| Content-Type | 是 (POST) | `application/json` |

### 签名算法

```
signature = HMAC-SHA256(secretKey, timestamp + "#" + memo + "#" + body)
```

- **GET 请求：** `body` 为查询参数的 JSON 格式，如 `{"symbol":"BTC/USD"}`
- **POST 请求：** `body` 为请求体 JSON

### 示例

```bash
curl -X GET 'https://api-cloud.bitmart.us/bm-us/api/v1/account' \
  -H 'X-BM-KEY: your_access_key' \
  -H 'X-BM-TIMESTAMP: 1702540800000' \
  -H 'X-BM-SIGN: your_signature'
```

---

## 响应格式

所有 API 响应遵循统一格式：

```json
{
  "code": 0,
  "message": "success",
  "data": { ... }
}
```

| 字段 | 类型 | 描述 |
|------|------|------|
| code | Integer | 响应码，`0` 表示成功 |
| message | String | 响应消息 |
| data | Object / Array | 响应数据；具体结构因接口而异（见各章节） |

---

## 行情接口（公开）

### 获取服务器时间

获取服务器时间戳，用于时间同步。

**接口：** `GET /api/v1/time`

**认证：** 不需要

**请求参数：** 无

**响应示例：**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "server_time": 1702540800000
  }
}
```

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| server_time | Long | 服务器时间戳（毫秒） |

---

### 获取交易所信息

获取所有交易对的详细信息，包括交易规则、精度限制等。

**接口：** `GET /api/v1/exchange-info`

**认证：** 不需要

**请求参数：** 无

**响应示例：**

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

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| messages | Array | 交易对列表 |
| messages[].product_id | String | 产品ID |
| messages[].symbol | String | 交易对符号，如 BTC/USD |
| messages[].description | String | 交易对描述 |
| messages[].image_url | String | 币种图标URL |
| messages[].fractional_qty_scale | String | 数量精度倍数（如 100000000 表示 8 位小数） |
| messages[].price_scale | String | 价格精度倍数（如 100 表示 2 位小数） |
| messages[].tick_size | String | 最小价格变动单位 |
| messages[].last_price | String | 最新成交价格 |
| messages[].change_percent | String | 24小时涨跌幅（百分比） |
| messages[].timestamp | String | 更新时间戳（毫秒） |
| messages[].minimum_trade_qty | String | 最小交易数量（已缩放） |
| messages[].price_limit | Object | 价格限制配置 |
| messages[].price_limit.relative_low | Number | 最低价比例（最低价 = 最新价 × relative_low） |
| messages[].price_limit.relative_high | Number | 最高价比例（最高价 = 最新价 × (1 + relative_high)） |
| messages[].order_size_limit | Object | 订单金额限制配置 |
| messages[].order_size_limit.total_notional_low | String | 最小订单金额（已缩放） |
| messages[].order_size_limit.total_notional_high | String | 最大订单金额（已缩放） |

**计算实际限制：**
- 最小订单金额：`total_notional_low / fractional_qty_scale / price_scale`
- 最大订单金额：`total_notional_high / fractional_qty_scale / price_scale`

---

### 获取最新价格

获取指定交易对或所有交易对的最新价格。

**接口：** `GET /api/v1/ticker/price`

**认证：** 不需要

**请求参数：**

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| symbol | String | 否 | 交易对（如 BTC/USD），不传则返回所有交易对 |

**响应示例：**

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

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| symbol | String | 交易对符号 |
| price | String | 最新价格 |
| change_percent | String | 24小时涨跌幅（百分比） |
| timestamp | String | 时间戳（毫秒） |

---

## 账户接口（私有）

### 获取账户信息

获取用户所有币种的余额信息。

**接口：** `GET /api/v1/account`

**认证：** 需要

**请求参数：** 无

**响应示例：**

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

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| balances | Array | 余额列表 |
| balances[].asset | String | 资产名称 |
| balances[].free | String | 可用余额 |
| balances[].locked | String | 冻结余额 |
| balances[].total | String | 总余额 |

---

### 获取指定币种余额

获取指定币种的余额详情。

**接口：** `GET /api/v1/balance`

**认证：** 需要

**请求参数：**

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| asset | String | 是 | 币种名称（如 BTC） |

**响应示例：**

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

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| asset | String | 资产名称 |
| free | String | 可用余额 |
| locked | String | 冻结余额 |
| total | String | 总余额 |

---

### 获取银行账户（ACH / 已绑定外部账户）

查询当前用户**已审核通过**的绑定银行账户（Plaid / 外部账户）。请将返回的 **`account_id`** 作为 **`POST /deposit/ach`**、**`POST /withdraw/ach`** 请求体中的 **`account_id`**。

**接口：** `GET /api/v1/account/bank-accounts`

**认证：** 需要

**请求参数：** 无

**响应示例：**

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "account_number": "****1234",
      "account_nickname": "Checking",
      "routing_name": "ACH",
      "account_holder_name": "Jane Doe",
      "account_type": "checking",
      "account_id": "17cbf404-bf52-4cc7-ad3d-6872ac5be5f4",
      "limit": 10000,
      "limit_type": null,
      "is_default": true
    }
  ]
}
```

**响应字段说明（`data` 为数组，元素字段）：**

| 字段 | 类型 | 描述 |
|------|------|------|
| account_number | String | 脱敏后的银行账号 |
| account_nickname | String | 账户昵称 |
| routing_name | String | 路由 / 通道展示名 |
| account_holder_name | String | 持有人姓名 |
| account_type | String | 账户类型（如 checking） |
| account_id | String | **外部账户 ID**，用于 ACH 入金/出金请求的 **`account_id`** |
| limit | Number | 限额（若有） |
| limit_type | Number | 限额类型等元数据（若有） |
| is_default | Boolean | 是否为默认账户 |

---

### 获取充币记录

获取用户的充币历史记录。

**接口：** `GET /api/v1/deposit/history`

**认证：** 需要

**请求参数：**

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| coin | String | 否 | 币种名称 |
| startTime | Long | 否 | 开始时间戳（毫秒） |
| endTime | Long | 否 | 结束时间戳（毫秒） |
| limit | Integer | 否 | 返回数量（默认 500，最大 1000） |

**响应示例：**

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

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| tx_id | String | 交易哈希 |
| asset | String | 资产名称 |
| amount | String | 充值数量 |
| address | String | 充值地址 |
| status | String | 状态：PENDING（待确认）、SUCCESS（成功）、FAILED（失败） |
| create_time | Long | 创建时间（毫秒） |
| complete_time | Long | 完成时间（毫秒） |

---

### 获取提币记录

获取用户的提币历史记录。

**接口：** `GET /api/v1/withdraw/history`

**认证：** 需要

**请求参数：**

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| coin | String | 否 | 币种名称 |
| startTime | Long | 否 | 开始时间戳（毫秒） |
| endTime | Long | 否 | 结束时间戳（毫秒） |
| limit | Integer | 否 | 返回数量（默认 500，最大 1000） |

**响应示例：**

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

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| tx_id | String | 交易哈希 |
| asset | String | 资产名称 |
| amount | String | 提币数量 |
| received_amount | String | 实际到账数量 |
| fee | String | 手续费 |
| address | String | 提币地址 |
| status | String | 状态：PENDING（待处理）、PROCESSING（处理中）、COMPLETED（已完成）、FAILED（失败）、CANCELED（已取消） |
| create_time | Long | 创建时间（毫秒） |
| complete_time | Long | 完成时间（毫秒） |

---

### 获取成交记录

获取用户的成交历史记录。

**接口：** `GET /api/v1/trades`

**认证：** 需要

**请求参数：**

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| symbol | String | 是 | 交易对（如 BTC/USD） |
| startTime | Long | 否 | 开始时间戳（毫秒） |
| endTime | Long | 否 | 结束时间戳（毫秒） |
| limit | Integer | 否 | 返回数量（默认 500，最大 1000） |

**响应示例：**

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

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| trade_id | String | 成交ID |
| order_id | String | 订单ID |
| symbol | String | 交易对 |
| price | String | 成交价格 |
| qty | String | 成交数量 |
| quote_qty | String | 成交金额 |
| commission | String | 手续费 |
| time | Long | 成交时间（毫秒） |
| is_buyer | Boolean | 是否为买方，true=买入，false=卖出 |

---

## 地址接口（私有）

### 获取地址列表

获取用户各币种的充币地址列表。

**接口：** `GET /api/v1/address`

**认证：** 需要

**请求参数：**

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| coin | String | 否 | 币种名称，不传则返回所有币种的地址 |

**响应示例：**

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

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| asset | String | 资产名称 |
| address | String | 充值地址 |

---

### 新增地址

为指定币种创建新的充币地址。

**接口：** `POST /api/v1/address`

**认证：** 需要

**请求参数：**

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| coin | String | 是 | 币种名称 |

**请求示例：**

```json
{
  "coin": "BTC"
}
```

**响应示例：**

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

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| asset | String | 资产名称 |
| address | String | 新创建的充值地址 |

---

## 充提接口（私有）

虚拟币**充值地址**、**可充值资产列表**（`queryAssets`）、**ACH** 入金/出金及记录查询、**虚拟币提现**。  
推荐串联：**`GET /account/bank-accounts`** 取 **`account_id`** → **`GET /queryAssets`** 取 **`asset`** → **`GET /deposit/address?asset=`** → **`POST /deposit/ach`** / **`POST /withdraw/ach`**。

**网关路径前缀（示例）：** `/bm-us/api/v1/...`，与账户类接口一致，以实际部署为准。

---

### 查询可充值资产列表

返回已展平的可充值资产列表（数据源与站内 **`GET /crypto/v1/queryAssets?transferType=deposit`** 一致）。列表项中的 **`asset`** 作为 **`GET /deposit/address`** 的查询参数 **`asset`**。

**接口：** `GET /api/v1/queryAssets`

**认证：** 需要（读权限；具体权限名以网关 / API Key 配置为准）

**请求参数：**

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| symbol | String | 否 | 可选筛选，例如 `BTC` |

**响应示例：**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [
      {
        "asset": "BTC"
      }
    ]
  }
}
```

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| list | Array | 展平后的资产行 |
| list[].asset | String | 展示用资产 / 网络符号；调用 **`GET /deposit/address`** 时作为查询参数 **`asset`** |

**规则说明：**

- 若底层数据含 **networks**，按网络拆行，**`asset`** 取对应 **`symbol`**。
- 若无 **networks**，**`asset`** 回退为 **`ticker`**。

---

### 获取虚拟币充值地址

根据 **`asset`** 查询参数返回链上充值 **`address`**（及可选 **`memo`**）。

**接口：** `GET /api/v1/deposit/address`

**认证：** 需要（如网关配置的虚拟币充值权限）

**请求参数：**

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| asset | String | 是 | 建议使用 `queryAssets` 返回的 **`asset`**（如 `BTC`）；也支持平台映射的其它展示名 |

**行为摘要：**

- **稳定币**（服务端配置，如 USDT/USDC）：通过**账户入金 RFQ** 能力获取充值地址（与站内法币 fund RFQ 一致），不走通用链上充值地址查询/创建分支。
- **其它资产**：先**查询**已有地址；若无则**创建**后返回。

**响应示例：**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "asset": "BTC",
    "address": "bc1q...",
    "memo": null
  }
}
```

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| asset | String | 展示用资产名（或与请求一致的归一化值） |
| address | String | 充值地址 |
| memo | String | 部分链需要的 memo / destination tag |

---

### 发起 ACH 入金

**接口：** `POST /api/v1/deposit/ach`

**认证：** 需要

**请求体（JSON）：**

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| client_order_id | String | 是 | 客户端幂等单号 |
| account_id | String | 是 | 来自 **`GET /account/bank-accounts`** 的 **`account_id`** |
| amount | Number | 是 | 金额 |
| currency | String | 否 | 默认 **USD** |

**请求示例：**

```json
{
  "client_order_id": "ach-dep-1702540800000",
  "account_id": "17cbf404-bf52-4cc7-ad3d-6872ac5be5f4",
  "amount": 10.00,
  "currency": "USD"
}
```

**响应示例：**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "transaction_id": "...",
    "status": "PENDING"
  }
}
```

| 字段 | 类型 | 描述 |
|------|------|------|
| transaction_id | String | 交易 / 订单引用 |
| status | String | 处理状态 |

---

### 发起 ACH 出金

**接口：** `POST /api/v1/withdraw/ach`

**认证：** 需要

**请求体（JSON）：** 与 ACH 入金相同（`client_order_id`、`account_id`、`amount`、`currency`）。

**响应：** 与 **`FiatTransferOpenApiResponse`** 一致（`transaction_id`、`status`）。

---

### 查询 ACH 入金记录

**接口：** `GET /api/v1/deposit/ach/history`

**认证：** 需要

**请求参数：**

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| page | Integer | 否 | 页码，默认 `1` |
| page_size | Integer | 否 | 每页条数，最大 **100** |
| limit | Integer | 否 | 与 `page_size` 等价别名（未传 `page_size` 时使用） |

**响应示例：**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [],
    "total": 0,
    "page": 1,
    "page_size": 20
  }
}
```

---

### 查询 ACH 出金记录

**接口：** `GET /api/v1/withdraw/ach/history`

**认证：** 需要

**请求参数：** 与 ACH 入金记录一致。

**响应：** 与 ACH 入金记录相同结构（`list`、`total`、`page`、`page_size`）。

---

### 发起虚拟币提现

**接口：** `POST /api/v1/withdraw/crypto`

**认证：** 需要

**请求体（JSON）：**

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| client_order_id | String | 是 | 客户端订单号 |
| coin | String | 是 | 币种 / 展示名（服务端映射底层资产） |
| address | String | 是 | 链上提现地址 |
| amount | Number | 是 | 提现数量 |
| memo | String | 否 | 需要时填写 memo |
| chain_full_code | String | 否 | 链 full code |
| chain_shot_code | String | 否 | 链短码 |
| withdrawal_quote_id | String | 否 | 若填写则走 **quote** 提现流程；不传则走 **withdrawNew** 流程 |

**请求示例：**

```json
{
  "client_order_id": "wd-crypto-1702540800000",
  "coin": "BTC",
  "address": "bc1q...",
  "amount": 0.0001
}
```

**响应示例：**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "withdrawal_request_id": "...",
    "status": "SUBMITTED"
  }
}
```

---

## 订单接口（私有）

### 创建订单

创建新的现货订单。

**接口：** `POST /api/v1/orders`

**认证：** 需要

**请求参数：**

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| symbol | String | 是 | 交易对（如 BTC/USD） |
| side | String | 是 | 交易方向：`BUY` 买入 / `SELL` 卖出 |
| type | String | 是 | 订单类型：`LIMIT` 限价单 / `MARKET` 市价单 |
| price | String | 条件必填 | 价格（限价单必填），已缩放值 |
| quantity | String | 条件必填 | 数量（已缩放值），限价单或市价单按数量下单时使用 |
| cash_order_qty | String | 条件必填 | 金额（已缩放值），市价单按金额下单时使用 |
| time_in_force | String | 否 | 有效期类型（默认：`TIME_IN_FORCE_GOOD_TILL_CANCEL`） |
| client_order_id | String | 否 | 自定义客户端订单ID |

**有效期类型说明：**

| 值 | 描述 |
|----|------|
| TIME_IN_FORCE_GOOD_TILL_CANCEL | 直到取消 |
| TIME_IN_FORCE_DAY | 当日有效 |
| TIME_IN_FORCE_IMMEDIATE_OR_CANCEL | 立即成交或取消 |
| TIME_IN_FORCE_FILL_OR_KILL | 全部成交或取消 |

**精度计算：**

所有价格和数量值在提交前必须乘以对应的缩放倍数：

```
缩放后价格 = 价格 × price_scale
缩放后数量 = 数量 × fractional_qty_scale
缩放后金额 = 金额 × fractional_qty_scale × price_scale
```

**价格范围验证（仅限价单）：**

```
最低价 = 最新价 × relative_low
最高价 = 最新价 × (1 + relative_high)
```

**订单金额验证：**

```
最小金额 = total_notional_low / fractional_qty_scale / price_scale
最大金额 = total_notional_high / fractional_qty_scale / price_scale
```

**请求示例 - 限价买入：**

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

**请求示例 - 市价买入（按金额）：**

```json
{
  "symbol": "BTC/USD",
  "side": "BUY",
  "type": "MARKET",
  "cash_order_qty": "100000000000",
  "time_in_force": "TIME_IN_FORCE_GOOD_TILL_CANCEL"
}
```

**响应示例：**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "order_id": "6WW511S0A021"
  }
}
```

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| order_id | String | 订单ID |

---

### 撤销订单

撤销指定订单。

**接口：** `POST /api/v1/orders/cancel`

**认证：** 需要

**请求参数：**

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| order_id | String | 是 | 订单ID |

**请求示例：**

```json
{
  "order_id": "6WW511S0A021"
}
```

**响应示例：**

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "result": true
  }
}
```

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| result | Boolean | 撤单结果，true=成功，false=失败 |

---

### 查询订单

根据订单ID查询订单详情。

**接口：** `GET /api/v1/orders`

**认证：** 需要

**请求参数：**

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| order_id | String | 是 | 订单ID |

**响应示例：**

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

**响应字段说明：**

| 字段 | 类型 | 描述 |
|------|------|------|
| order_id | String | 订单ID |
| symbol | String | 交易对 |
| side | String | 交易方向：BUY=买入，SELL=卖出 |
| type | String | 订单类型：LIMIT=限价单，MARKET=市价单 |
| price | String | 委托价格（已缩放） |
| order_qty | String | 委托数量（已缩放） |
| filled_qty | String | 已成交数量（已缩放） |
| avg_price | String | 成交均价（已缩放） |
| status | String | 订单状态 |
| create_time | String | 创建时间（ISO 8601格式） |

**订单状态说明：**

| 状态 | 描述 |
|------|------|
| NEW | 新订单 |
| PARTIALLY_FILLED | 部分成交 |
| FILLED | 完全成交 |
| CANCELED | 已撤销 |
| REJECTED | 已拒绝 |

---

## 错误码

| 错误码 | 描述 |
|--------|------|
| 0 | 成功 |
| 1001 | 参数错误 |
| 1002 | 用户不存在 |
| 2001 | 余额不足 |
| 2002 | 订单不存在 |
| 2003 | 价格超出范围 |
| 2004 | 订单金额超出范围 |
| 3001 | 系统错误 |

---

## 频率限制

- **公开接口：** 10 次/秒（按 IP）
- **私有接口：** 20 次/秒（按 API Key）
- **订单接口：** 5 次/秒（按 API Key）

---

## 更新日志

| 版本 | 日期 | 变更说明 |
|------|------|----------|
| v1.0.0 | 2025-12-19 | 初始版本 |
| v1.1.0 | 2026-04-15 | 新增充提接口（`queryAssets`、`deposit/address`、ACH 与虚拟币提现）及 `account/bank-accounts` |
