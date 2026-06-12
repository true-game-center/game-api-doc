# 用户游戏次数接口

**Last Updated:** 2026-06-12

## 接口信息

- **Method:** `POST`
- **Path:** `/oapi/game-order-collection/user-game-play-count`
- **Content-Type:** `application/json`

## 鉴权

所有 `/oapi/**` 请求都需要在请求头中携带 `token`。

| Header | Required | Description |
| --- | --- | --- |
| token | Yes | JWT token |

## 请求参数

```json
{
  "userId": 210048,
  "gameIds": [1497254068404215812, 1497254068404215813],
  "beginTime": 1749657600000,
  "endTime": 1749743999999
}
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| userId | Long | Yes | 用户 ID；也支持别名 `userid`、`user_id` |
| gameIds | Long[] | Yes | 游戏 ID 列表，最多 100 个；也支持别名 `game_ids` |
| beginTime | Long | Yes | 开始时间，毫秒时间戳 |
| endTime | Long | Yes | 结束时间，毫秒时间戳 |

## 统计口径

| Field | Rule |
| --- | --- |
| 时间范围 | 按 `game_order_collection.bet_time` 过滤，包含开始和结束时间 |
| playCount | `count(*)`，按 `game_id` 聚合 |
| 返回范围 | 请求中的每个去重后 `gameId` 都返回一条；没有匹配订单时 `playCount=0` |
| 返回顺序 | 与请求 `gameIds` 去重后的顺序一致 |

## 响应示例

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success",
  "data": [
    {
      "gameId": 1497254068404215812,
      "playCount": 3
    },
    {
      "gameId": 1497254068404215813,
      "playCount": 0
    }
  ]
}
```
