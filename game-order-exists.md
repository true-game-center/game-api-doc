# 用户是否下过注接口

**Last Updated:** 2026-06-10

该接口用于查询当前登录用户是否已经产生过游戏订单，即是否下过注。

## 接口信息

- **Method:** `GET`
- **Path:** `/game-center/game/order/exists`
- **Gateway Path:** `/game/order/exists`（当网关已剥离 `/game-center` 前缀时使用）

## 鉴权

请求头必须携带用户登录 `token`。接口只根据 `token` 解析出的当前用户查询，不接收 `userId` 请求参数。

| Header | Required | Description |
| --- | --- | --- |
| token | Yes | 用户登录 token |

## 请求参数

无。

## 判定口径

| Field | Rule |
| --- | --- |
| 用户范围 | 使用 `token` 中解析出的当前用户 ID |
| 数据来源 | `game_order_collection` |
| 判定条件 | 存在任意 `user_id = 当前用户 ID` 的订单记录即视为下过注 |
| 过滤条件 | 不按游戏、币种、订单状态或时间过滤 |

## 响应示例

已下过注：

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success",
  "data": 1
}
```

未下过注：

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success",
  "data": 0
}
```

## 返回值说明

| Field | Type | Description |
| --- | --- | --- |
| data | Integer | `1` 表示当前用户下过注；`0` 表示当前用户未下过注 |

`token` 缺失、无效或解析失败时，按 Game Center 统一鉴权错误返回。
