# 用户下注存在性 OAPI

**Last Updated:** 2026-06-16

该接口用于后台或其他服务按 `userId` 查询用户是否已经产生过游戏订单，并返回最近一次下注时间。

## 接口信息

- **Method:** `GET`
- **Path:** `/oapi/game-order-collection/exists-order`

## 鉴权

所有 `/oapi/**` 请求都需要在请求头中携带 `token`。

| Header | Required | Description |
| --- | --- | --- |
| token | Yes | JWT token |

## 请求参数

| Field | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| userId | query | Long | Yes | 用户 ID |

## 判定口径

| Field | Rule |
| --- | --- |
| 用户范围 | 使用 query 参数 `userId` |
| 数据来源 | `game_order_collection` |
| 判定条件 | 存在任意 `user_id = userId` 的订单记录即视为下过注 |
| 最近下注时间 | 返回该用户订单中的最大 `bet_time` |
| 过滤条件 | 不按游戏、币种、订单状态或时间过滤 |

## 响应示例

已下过注：

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success",
  "data": {
    "existed": 1,
    "betTime": 1782549057000
  }
}
```

未下过注：

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success",
  "data": {
    "existed": 0,
    "betTime": null
  }
}
```

## 返回值说明

| Field | Type | Description |
| --- | --- | --- |
| data.existed | Integer | `1` 表示该用户下过注；`0` 表示该用户未下过注 |
| data.betTime | Date / null | 最近一次下注时间；`existed=0` 时为 `null` |

`userId` 缺失或查询失败时，按 Game Center 统一错误结构返回。
