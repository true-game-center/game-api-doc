# 用户游戏任务统计接口

**Last Updated:** 2026-06-05

## 接口信息

- **Method:** `POST`
- **Path:** `/oapi/game-order-collection/getUserGameTaskStat`
- **Content-Type:** `application/json`

## 鉴权

所有 `/oapi/**` 请求都需要在请求头中携带 `token`。

| Header | Required | Description |
| --- | --- | --- |
| token | Yes | JWT token |

## 请求参数

```json
{
  "kolUserId": 11012184,
  "userId": 11011234,
  "gameId": 1497254068404215812,
  "beginTime": 1780488000000,
  "endTime": 1780574399000
}
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| kolUserId | Long | Yes | A0 用户 ID；也支持别名 `a0` |
| userId | Long | Yes | 用户 ID；也支持别名 `userid`、`user_id` |
| gameId | Long | Yes | 游戏 ID；也支持 `gameid`、`game_id` |
| beginTime | Long | Yes | 开始时间，毫秒时间戳 |
| endTime | Long | Yes | 结束时间，毫秒时间戳 |

## 统计口径

| Field | Rule |
| --- | --- |
| 时间范围 | 按 `game_order_collection.bet_time` 过滤，包含开始和结束时间 |
| settleScore | `sum(settle_score)`，按 A0 currency unit 换算后返回 |
| betScore | `sum(bet_score)`，按 A0 currency unit 换算后返回 |
| gameWinCount | `count(order_status = 1)` |

## 响应示例

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success",
  "data": {
    "kolUserId": 11012184,
    "userId": 11011234,
    "gameId": 1497254068404215812,
    "settleScore": 120.50000000,
    "betScore": 100.00000000,
    "gameWinCount": 3
  }
}
```

没有匹配订单时，金额字段返回 `0`，`gameWinCount` 返回 `0`。
