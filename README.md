# Game Center Integration Guide

**Target Audience:** Client developers, Server developers, QA<br>
**Last Updated:** 2026-06-10<br>
**Document Version:** V1.3.4

---

## 1. Introduction

This document describes the integration between the game side and Game Center (GC) via standardized APIs. Main features include:

- Get game URL
- Login & authentication
- Get user info and balance
- Deduct / credit points
- Game record query
- A0 service config management

## 2. Interaction Sequence Diagram

![Interaction Sequence Diagram](./pp.png)

## 3. Integration Flow Overview

1. **Get game URL:** Game Center requests the game URL from the game server using the game id and user login token.
2. **Open game & verify:** The user opens the game via the URL; the game server must call Game Center to validate the user token and obtain username and avatar.
3. **Get balance:** The game server requests the user balance from Game Center.
4. **Credit operations:** When the user plays for points, the game server sends deduct/credit requests to Game Center.
5. **Exit game:** When the user exits the game, the game server sends an exit request to Game Center.

---

## Related Documents

- [A0 Game Service Config API](./game-a0-service-config.md)
- [Game Supplier Management API](./game-supplier.md)
- [User Bet Existence API](./game-order-exists.md)
- [User Game Play Count API](./game-order-collection-user-game-play-count.md)

---

<a id="order-linking-ids"></a>
## 订单串联标识（Round / Transaction / Tick）

> **第一阶段（Phase 1）。** 本节在现有模型基础上做最小扩展，目的是把「下注」与「派彩」串联起来、让对账闭环，暂不引入会话级（session）追踪，详见下方[后续规划](#roadmap-phase-2)。

当前对接已携带 `transactionId`（在 `user.score.change` 上）以及结算后产生的 `orderId`。缺口在于：下注那一刻没有任何字段把这笔**下注**和它后续的**派彩/结算**关联起来，且钱包侧的 `transactionId` 没有回传到订单查询/采集接口的响应里，导致两边对不上账。第一阶段用「三个新增字段 + 一处暴露」补齐：

| 字段 | 现状 | 第一阶段 |
|------|------|----------|
| `roundId` | ❌ 只有结算后才产生的 `orderId` | **新增**（可选）。一个回合 / 一次旋转 / 一手牌 / 一张注单。同一回合的扣款与其后续派彩/退款共用同一个 `roundId`，是对账锚点；`orderId` 是某个 `roundId` 结算后的订单视图。 |
| `tickId` | ❌ 无 | **新增**（可选）。一笔交易 / 一个回合内的最细子事件（如捕鱼每一炮、连续游戏的每一步）。仅 tick 型游戏需要，普通 slot 可不传。 |
| `transactionId` | ✅ 仅 `score.change` 有 | **暴露**到 `order.query`（5.2）与 `order.collect`（5.5）响应中，使钱包流水与订单记录可对上。仍是幂等键——同一个值不得重复扣费。 |
| `refTransactionId` | ⚠️ 只有 `opType: return`，没有指回原单 | **新增**（可选）。`return` 携带其所冲正的原 `transactionId`。账本只追加（append-only），纠错用反向流水，绝不原地修改。 |
| `groupId` | ✅ 已有 | 不变 |
| 业务细节 | ✅ 走 `ext`（透传） | 不变——体育投注项、彩票开奖号、slot 转轴、jackpot 档位等继续放 `ext`，不新增顶层字段 |

所有新增字段均为**可选**：尚未产出 `roundId` 的供应商可照旧运行。供应商应优先在扣款和派彩两侧带上同一个稳定的 `roundId`。

<a id="roadmap-phase-2"></a>
**后续规划（Phase 2，暂未实现）：** 在 `roundId` 之上再叠一层 `sessionId`（登录会话）→ `gameSessionId`（单游戏会话）的包裹。该层需要会话追踪基础设施，待有明确需求再上，避免现在就强制所有供应商接入。

---

## 4. Game Center API (Public Category)

### 4.1 User Token Validation

- **Method:** `POST`
- **Path:** `/game-proxy/user/token/check`

**Request Headers**

| Parameter | Description |
|-----------|-------------|
| token     | 70d79fa3-e2f4-46ae-9b28-bbee4cd795b3 (Token secret: game-proxy-10086) |

**Request Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "user.session.check",
  "params": {
    "token": "sdfsdfsdfsdfss"  // User token from frontend
  }
}
```

**Response Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "user.session.check",
  "result": {
    "a0": 100841,  // Root agent
    "userId": 100821,  // User ID
    "avatar": "https:// pbtest.buyacard.cc/img/user/avatar/a.png",  // Avatar
    "nickname": "",  // Nickname
    "gender": 1  // Gender: 0 Unknown, 1 Male, 2 Female
  },
  "error": null
}
```

(gender: 0 Unknown / 1 Male / 2 Female)

---

### 4.2 User Score Change

- **Method:** `POST`
- **Path:** `/game-proxy/user/score/change`

**Request Headers**

| Parameter | Description |
|-----------|-------------|
| token     | 70d79fa3-e2f4-46ae-9b28-bbee4cd795b3 |

**Request Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "user.score.change",
  "params": {
    "a0": 1008171,  // Root agent
    "userId": 1019818,  // User ID
    "category": "0",  // Game category ID
    "providerId": "1",  // Game provider ID
    "gameId": "181818",  // Game ID
    "roundId": "R-77f0a1",  // 回合 ID，串联本次下注与其后续派彩/退款；可选，建议传
    "cost": 10.00, // cost value
    "win": 999.00,  // return value
    "score": 999.00,  // Score/points, if cost step, it equals to costt, if settle step it equals to win.
    "transactionId": "23424324324",  // 交易 ID，幂等键，同一值不得重复扣费
    "refTransactionId": "23424324300",  // 本条所冲正的原 transactionId；opType=return 时必填，否则省略
    "tickId": "T-001",  // tick 子事件 ID，最细粒度；可选，仅 tick 型游戏（如捕鱼）需要
    "opType": "debit",  // debit / credit / return (cancel-refund) / gift
    "currency": "GOLD",  // Currency: GOLD=free, VND/USD=real money, FAM=family coin
    "dataset": "default",  // Passthrough dataset to the A0 score service, optional
    "enableJackpot": 1,  // Enable JackPot: 0=No 1=Yes, optional
    "ext": "||1499486369695664870|74|||null_4030004|"  // Passthrough string to the A0 score service, optional
  }
}
```

(opType: debit / credit / return for cancel-refund / gift for direct bonus credit. `gift` is mapped to internal opType `3`; `score` must be greater than 0.)

`dataset`, `enableJackpot`, and `ext` are optional passthrough fields. Game Center forwards them unchanged to the downstream `/oapi/score/cost` request after routing by `a0`/`kolUserId` and `BALANCE` service configuration.

`roundId`、`refTransactionId`、`tickId` 属于[订单串联标识](#order-linking-ids)：同一回合的扣款（`opType: debit`）与其后续派彩（`opType: credit`）必须携带相同的 `roundId`；`return` 必须把 `refTransactionId` 设为它所冲正的 `transactionId`。`tickId` 仅 tick 型游戏需要。

**Response Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "user.score.change",
  "result": {
    "a0": 100841,
    "userId": 100821,  // User ID
    "currency": "GOLD",  // Currency; FAM=family coin
    "score": "100.00"  // Amount
  },
  "error": null
}
```

---

### 4.3 User Exit

- **Method:** `POST`
- **Path:** `/game-proxy/user/game/exit`

**Request Headers**

| Parameter | Description |
|-----------|-------------|
| token     | 70d79fa3-e2f4-46ae-9b28-bbee4cd795b3 |

**Request Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "user.exit",
  "params": {
    "a0": 1008171,  // Root agent
    "userId": 1019818,  // User ID
    "category": "0",  // Game category ID
    "providerId": "1",  // Provider ID
    "gameId": "181818"  // Game ID
  }
}
```

**Response Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "user.exit",
  "result": true,
  "error": null
}
```

---

### 4.4 Get User Balance

- **Method:** `POST`
- **Path:** `/game-proxy/user/balance`

**Request Headers**

| Parameter | Description |
|-----------|-------------|
| token     | jwt |

**Request Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "user.balance",
  "params": {
    "a0": "1111111",  // Root agent
    "userId": 123123,  // User ID
    "currency": "GOLD",  // Currency: GOLD=free, VND/USD=real money, FAM=family coin
    "gameInfo": {} // gameInfo object
  }
}
```

**Response Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "user.balance",
  "result": {
    "a0": 100841,  // Root agent
    "userId": 100821,  // User ID
    "currency": "GOLD",  // Currency: GOLD=free, VND/USD=real money, FAM=family coin
    "score": "100.00"  // Amount
  },
  "error": null
}
```

---

### 4.5 Recent User Game Win/Loss Statistics

- **Method:** `POST`
- **Path:** `/oapi/game-order-collection/getUserRecentGameWinLossStat`

Returns the last 5 games played by a user from `game_order_collection`, then groups each game's settled records into win and lose buckets. Each bucket includes the net payout difference for the last 15 minutes and last 24 hours.

**Calculation Rules**

| Field | Rule |
|-------|------|
| Last played games | Top 5 distinct `game_id` ordered by `MAX(bet_time)` desc |
| Win bucket | `order_status = 1` |
| Lose bucket | `order_status = 3` |
| Time windows | Based on `bet_time`, relative to request processing time |
| Difference amount | `settle_score - bet_score`; converted by the A0 currency unit before summing |

**Request Body**

```json
{
  "a0": 11012184,
  "userId": 11011234
}
```

`kolUserId` can be used instead of `a0`. `userid` and `user_id` are also accepted as aliases for `userId`.

**Response Body**

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success",
  "data": {
    "games": [
      {
        "gameId": 1497254068404215812,
        "lastBetTime": "2026-05-27T17:30:57",
        "win": {
          "last15MinutesDiffAmount": 110.53100000,
          "last24HoursDiffAmount": 322.78800000
        },
        "lose": {
          "last15MinutesDiffAmount": -2.47100000,
          "last24HoursDiffAmount": -9.30400000
        }
      }
    ]
  }
}
```

If a game has no settled win or lose records in a window, the corresponding amount is `0`.

---

## 5. Game-Side APIs

The following APIs are implemented and exposed by the game side for Game Center to call.

### 5.1 Query Game Statistics Interface

- **Method:** `POST`
- **Path:** `/game-proxy/game/order/stat`

**Request Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "game.order.stat",
  "params": {
    "a0": 19191,  // Root agent, required
    "userIds": [199817, 12123],  // User IDs, optional
    "currency": "GOLD",  // Currency, required; FAM=family coin
    "isSettle": 0,  // Is settled: 0=No 1=Yes, optional
    "state": [],  // State 0,1,2,3,4,5,7, optional
    "gameInfo": {
        "category": "0",  // Game category ID, optional
        "providerId": "1",  // Game provider ID, required
        "gameId": "101891"  // Game ID, optional
    },
    "groupId": "DEF",  // groupId, default DEF, optional
    "beginTime": "YYYY-MM-DD HH:mm:ss",  // Start time, required
    "endTime": "YYYY-MM-DD HH:mm:ss",  // End time, required
    "timezone": 0,  // 0=Singapore 1=US Eastern, required
    "page": 1,
    "size": 100,
    "hasStat": 0  // Need total: 0=No 1=Yes, required
  }
}
```

**Response Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "result": {
    "total": 100,  // Total count
    "totalBetScore": 0,  // Total bet amount
    "totalUserCount": 0,  // Distinct user count
    "totalSettleScore": 0,  // Total settle amount
    "totalValidScore": 0,  // Total valid bet amount
    "list": [
      {
        "gameId": "1",  // Game ID
        "groupId": "1",  // Room/group ID
        "currency": "RMB",  // Currency; FAM=family coin
        "betScore": 100.0,  // Bet amount
        "settleScore": 100.0,  // Payout amount
        "validScore": 100.0,  // Valid bet
        "userId": 100211,  // User ID
        "recordCount": 2  // Number of original orders aggregated into this grouped row (group key: userId|gameId|groupId|currency); must be > 0 when the group has orders
      }
    ]
  },
  "error": null
}
```

---

### 5.2 Query Game Order Interface

- **Method:** `POST`
- **Path:** `/game-proxy/game/order/query`

**Request Headers**

| Parameter | Description |
|-----------|-------------|
| token     | 70d79fa3-e2f4-46ae-9b28-bbee4cd795b3 |

**Request Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "game.order.query",
  "params": {
    "orderId": "1213123213",  // Order ID, optional
    "a0": 19191,  // Root node ID, required
    "userId": 199817,  // User ID, optional
    "currency": "GOLD",  // Currency, required; FAM=family coin
    "state": [0],  // State: 0 Unsettled, 1 Win, 2 Draw, 3 Loss, 4 User Cancelled, 5 System Cancelled, 7 Abnormal, optional
    "gameInfo": {
        "category": "0",  // Game category ID, optional
        "providerId": "1",  // Game provider ID, required
        "gameId": "101891"  // Game ID, optional
    },
    "returnGameInfo": 0,  // Return order detail: 0=No 1=Yes, required
    "groupId": "sdfs",  // groupId, default DEF, optional
    "beginTime": "YYYY-MM-DD HH:mm:ss",  // Start time, required
    "endTime": "YYYY-MM-DD HH:mm:ss",  // End time, required
    "timeZone": 0,  // 0=Singapore 1=US Eastern, required
    "page": 1,
    "size": 100,
    "hasStat": 0  // Need total: 0=No 1=Yes, required
  }
}
```

**Response Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "result": {
    "total": 100,  // Total count
    "totalBetScore": 0,  // Total bet amount
    "totalUserCount": 0,  // Distinct user count
    "totalSettleScore": 0,  // Total settle amount
    "totalValidScore": 0,  // Total valid bet amount
    "list": [
      {
        "a0": "12313",
        "userId": "324324",  // User ID
        "gameId": "1",  // Game ID
        "orderId": "2342342",  // Order ID（某个 roundId 结算后的订单视图）
        "roundId": "R-77f0a1",  // 回合 ID，下注与结算共用；可选，proxy 不支持时可为 null
        "transactionId": "23424324324",  // 结算交易 ID；可选，proxy 不支持时可为 null
        "tickId": "T-001",  // tick 子事件 ID；可选，proxy 不支持时可为 null
        "currency": "GOLD",  // Currency; FAM=family coin
        "betTime": "YYYY-MM-DD HH:mm:ss",  // Bet time
        "state": 1,  // State: 0 Unsettled, 1 Win, 2 Draw, 3 Loss, 4 User Cancelled, 5 System Cancelled, 7 Abnormal
        "betScore": 100.0,  // Bet amount
        "settleScore": 100.0,  // Payout amount
        "validScore": 100.0,  // Valid bet
        "settleTime": "YYYY-MM-DD HH:mm:ss",  // Settle time
        "groupId": "2322",  // Group ID
        "isSettle": 0,  // Is settled: 0=No 1=Yes (state 1,2,3)
        "gameInfo": {},
        "dataset": "default",  // Passthrough dataset from enter game, optional, can be null when proxy does not support it
        "enableJackpot": 1,  // Enable JackPot: 0=No 1=Yes, optional, can be null when proxy does not support it
        "ext": "||1499486369695664870|74|||null_4030004|"  // Passthrough string from enter game, optional, can be null when proxy does not support it
      }
    ]
  },
  "error": null
}
```

(state: 0 Unsettled / 1 Win / 2 Draw / 3 Loss / 4 User Cancelled / 5 System Cancelled / 7 Abnormal)

---

### 5.3 Query Order Detail Interface (Deprecated)

- **Method:** `POST`
- **Path:** `/game-proxy/game/order/detail`

> Deprecated. This interface returns the order detail URL string. Do not use it for new integrations.

**Request Headers**

| Parameter | Description |
|-----------|-------------|
| token     | 70d79fa3-e2f4-46ae-9b28-bbee4cd795b3 |

**Request Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "game.order.detail",
  "params": {
    "orderId": "1213123213",
    "language": "CN",
    "currency": "CNY",  // Currency; FAM=family coin
    "gameInfo": { "category": "0", "providerId": "1", "gameId": "101891" }
  }
}
```

**Response Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "result": "https://sfdsfd.com",  // Order detail URL
  "error": null
}
```

---

### 5.4 Get Game URL

- **Method:** `POST`
- **Path:** `/game-proxy/game/url`

**Request Headers**

| Parameter    | Description |
|--------------|-------------|
| Content-Type | application/json |
| token        | 70d79fa3-e2f4-46ae-9b28-bbee4cd795b3 (See: https://pb-api-doc.pwtk.cc/project/253/wiki) |

**Request Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "game.url",
  "params": {
    "token": "sdfdsfdsfsd",  // User token, required
    "device": "PC",  // Terminal, required: PC, H5
    "language": "zh-CN",  // Language, required
    "currency": "GOLD",  // Currency, required; FAM=family coin
    "groupId": "1000",  // Group ID, default DEF, optional; required when isGroupTrace=1
    "isGroupTrace": 0,  // 0=No tracking (default), 1=Tracking (must set groupId)
    "isDemo": 0,  // Is demo: 0=No 1=Yes, required
    "limitConfigId": co2, // 现红配置，no required
    "dataset": "default", // 透传给游戏服务方的数据集，no required
    "enableJackpot": 1, // 是否启用 JackPot：0 不启用，1 启用，no required
    "ulc": "02", // 用户生命周期代码，透传给游戏服务方，no required
    "ext": "||1499486369695664870|74|||null_4030004|", // 透传字符串参数，no required
    "gameInfo": {
        "category": "0",  // Game category ID, required
        "providerId": "1494326540744264616",  // 1494326540744264616 is h5games，Game provider ID, required
        "gameId": "101891"  // Game ID, optional; 6 games: 1503790445774242709 carnival, 1503780287216092019 foddie, 1503779730992661356 royal, 1499486369695664870 tanmin, 1499484616333986513 farm, 1494327065715936934 转盘
    },
    "returnUrl": ""  // Return URL, optional
  }
}
```

#### `ulc` Code Definition

| User Lifecycle | Code |
|----------------|------|
| D0-D2: 新用户 | `02` |
| D3-D6: 次新用户 | `36` |
| D7-D14: 成长期用户 | `714` |
| D15-D30: 稳定期用户 | `1530` |
| D31+: 成熟用户 | `3100` |

**Response Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "result": {
    "url": "https://www.ddf.com",  // Game entry URL
    "config": {}
  },
  "error": null
}
```

---

### 5.5 Order Collection Interface

- **Method:** `POST`
- **Path:** `/game-proxy/game/order/collect`

**Request Headers**

| Parameter    | Description         |
|--------------|---------------------|
| Content-Type | application/json    |

**Request Body**

Request format is JSON-RPC. Example:

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "game.order.collect",
  "params": {
    "startTime": "YYYY-MM-DD HH:mm:ss",
    "endTime": "YYYY-MM-DD HH:mm:ss",
    "pageNum": 1,
    "pageSize": 100,
    "providerId": "1",
    "categoryId": "0",
    "gameId": "101891",
    "timeZone": 0
  }
}
```

| Parameter   | Type   | Description                              |
|-------------|--------|------------------------------------------|
| startTime   | string | Start time                               |
| endTime     | string | End time                                 |
| pageNum     | number | Page number                              |
| pageSize    | number | Page size                                |
| providerId  | string | Game provider ID (optional)              |
| categoryId  | string | Game category ID (optional)              |
| gameId      | string | Game ID (optional)                       |
| timeZone    | number | Timezone: 0 Beijing Time / 1 US Eastern  |

**Response Body**

Returns paginated data. Example:

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "result": {
    "total": 100,
    "list": [
      {
        "a0": "string",
        "userId": "string",
        "providerId": "string",
        "categoryId": "string",
        "gameId": "string",
        "groupId": "string",
        "orderId": "string",
        "roundId": "string",
        "transactionId": "string",
        "tickId": "string",
        "betScore": "string",
        "effectScore": "string",
        "settleScore": "string",
        "status": "string",
        "betTime": "string",
        "settleTime": "string",
        "gameInfo": "string",
        "currency": "string"  // Currency; FAM=family coin
      }
    ]
  },
  "error": null
}
```

| Field        | Description    |
|--------------|----------------|
| total          | Total count    |
| list           | Order list     |
| roundId        | 回合 ID，下注与结算共用 |
| transactionId  | 结算交易 ID |
| tickId         | tick 子事件 ID |
| betScore       | Bet amount     |
| effectScore    | Valid bet      |
| settleScore    | Settle amount  |
| status         | Order status   |
| currency       | Currency; FAM=family coin |

---

### 5.6 Exit Game

- **Method:** `POST`
- **Path:** `/game-proxy/game/exit`

**Request Headers**

| Parameter | Description |
|-----------|-------------|
| token     | 70d79fa3-e2f4-46ae-9b28-bbee4cd795b3 |

**Request Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "game.user.exit",
  "params": {
    "a0": 19191,
    "userId": 199817,
    "device": "PC",
    "language": "CN",
    "currency": "RNB",  // Currency; FAM=family coin
    "groupId": "were",
    "gameInfo": { "category": "0", "providerId": "1", "gameId": "101891" }
  }
}
```

**Response Body**

```json
{
  "jsonrpc": "2.0",
  "id": "req-001",
  "result": true,
  "error": null
}
```

---

## 6. Request Authentication: JWT Token

API requests must carry a JWT Token in the request header for authentication. The caller must add a `token` field in **Request Headers** with a valid JWT Token. The server will verify the token; only valid tokens are allowed to access the APIs.

### Token Generation Demo

```java
public String getToken() {
    Map<String, String> map = new HashMap<>(2);
    map.put("service", "game-center");
    map.put("exp", System.currentTimeMillis() + 24 * 3600 * 1000L + "");
    // map.put("iat", System.currentTimeMillis() + "");
    return JWTUtil.generateToken(key, System.currentTimeMillis() + 24 * 3600 * 1000L, map);
}
```
