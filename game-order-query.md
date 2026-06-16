# 游戏订单查询 OAPI

**Last Updated:** 2026-06-16

该接口从 `game_order_collection` 查询游戏订单，支持下注时间和结算时间过滤，并可按需返回统计数据。

## 接口信息

- **Method:** `POST`
- **Path:** `/oapi/game-order/query`
- **Content-Type:** `application/json`

## 鉴权

所有 `/oapi/**` 请求都需要在请求头中携带 `token`。

| Header | Required | Description |
| --- | --- | --- |
| token | Yes | JWT token |
| language | No | 游戏名称国际化语言；传入后用于按语言匹配或返回游戏名称 |

## Query 参数

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| kolUserId | Long | No | 当 `fromCm=true` 时必填，并会覆盖 body 中的 `kolUserId` |
| fromCm | Boolean | No | `true` 表示来自 CM/proxy 场景，只允许查询 query 参数 `kolUserId` 下的数据；缺少 `kolUserId` 时返回空结果 |

## 请求参数

```json
{
  "kolUserId": 11012184,
  "userId": 210048,
  "userIdList": [210048, 210049],
  "orderId": "ORD-001",
  "gameId": 1497254068404215812,
  "gameName": "Fortune",
  "gameCategoryId": 1486042272511296851,
  "gameSupplierId": 1491818780504883931,
  "state": 1,
  "beginTime": "2026-06-01 00:00:00",
  "endTime": "2026-06-16 23:59:59",
  "settleBeginTime": "2026-06-10 00:00:00",
  "settleEndTime": "2026-06-16 23:59:59",
  "currency": "GOLD",
  "language": "en-US",
  "isGetDetail": 1,
  "timeZone": 0,
  "hasStat": 1,
  "page": 1,
  "size": 20
}
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| kolUserId | Long | No | A0/KOL 用户 ID |
| userId | Long | No | 单个用户 ID |
| userIdList | Long[] | No | 批量用户 ID 过滤 |
| orderId | String | No | 订单 ID |
| gameId | Long | No | 游戏 ID |
| gameName | String | No | 游戏名称模糊匹配；会结合 `language` 优先查对应语言名称 |
| gameCategoryId | Long | No | 游戏分类 ID |
| gameSupplierId | Long | No | 游戏供应商 ID |
| state | Integer | No | 订单状态：`0` 未结算，`1` 赢，`2` 和，`3` 输，`4` 用户取消，`5` 系统取消，`7` 异常 |
| beginTime | String | No | 下注开始时间，格式 `yyyy-MM-dd HH:mm:ss`，过滤 `bet_time >= beginTime` |
| endTime | String | No | 下注结束时间，格式 `yyyy-MM-dd HH:mm:ss`，过滤 `bet_time <= endTime` |
| createTimeStart | Long | No | 下注开始时间毫秒时间戳；传入后会转换并覆盖 `beginTime` |
| createTimeEnd | Long | No | 下注结束时间毫秒时间戳；传入后会转换并覆盖 `endTime` |
| settleBeginTime | String | No | 结算开始时间，格式 `yyyy-MM-dd HH:mm:ss`，过滤 `settle_time >= settleBeginTime` |
| settleEndTime | String | No | 结算结束时间，格式 `yyyy-MM-dd HH:mm:ss`，过滤 `settle_time <= settleEndTime` |
| settleTimeStart | Long | No | 结算开始时间毫秒时间戳；传入后会转换并覆盖 `settleBeginTime` |
| settleTimeEnd | Long | No | 结算结束时间毫秒时间戳；传入后会转换并覆盖 `settleEndTime` |
| currency | String | No | 币种；传入后可启用统计金额查询 |
| language | String | No | Body 级语言参数；Header 中的 `language` 会覆盖该值 |
| isGetDetail | Integer | No | 是否返回订单详情：`0` 否，`1` 是 |
| timeZone | Integer | No | 时间戳转换时区：`0` 北京/新加坡，`1` 美东，`2` 不转换场景 |
| timeType | Integer | No | 当未传 `beginTime` 时可用：`0` 最近 1 天，`1` 最近 3 天，`3` 最近 7 天，`4` 最近 30 天 |
| hasStat | Integer | No | 是否查询统计：`0` 否，`1` 是；仅当 `currency` 非空时返回金额统计 |
| page | Integer | No | 页码 |
| size | Integer | No | 每页数量 |

## 结算时间过滤

结算时间过滤同时支持字符串和毫秒时间戳两种入参：

- `settleBeginTime` / `settleEndTime`：直接传业务时间字符串。
- `settleTimeStart` / `settleTimeEnd`：传毫秒时间戳，服务会根据 `timeZone` 转换为 `settleBeginTime` / `settleEndTime`。

结算时间过滤会同时作用于列表查询、总数查询和统计查询。

## 响应示例

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success",
  "data": {
    "total": 2,
    "totalBetScore": 300.00000000,
    "totalSettleScore": 260.00000000,
    "totalValidScore": 300.00000000,
    "totalUserCount": 1,
    "list": [
      {
        "kolUserId": "11012184",
        "userId": "210048",
        "gameId": "1497254068404215812",
        "gameName": "Fortune",
        "gameSupplierId": "1491818780504883931",
        "gameSupplierCode": "jili",
        "gameSupplierName": "JILI",
        "gameCategoryId": "1486042272511296851",
        "gameCategoryName": "Slot",
        "orderId": "ORD-001",
        "currency": "GOLD",
        "state": 1,
        "betTime": 1782549057000,
        "betScore": 100.00000000,
        "settleScore": 160.00000000,
        "netScore": 60.00000000,
        "validScore": 100.00000000,
        "settleTime": 1782549657000,
        "groupId": "DEF",
        "isSettle": 1,
        "dataset": "default",
        "enableJackpot": 1,
        "ext": "{}",
        "gameInfo": {},
        "isOrderDetail": 1
      }
    ]
  }
}
```

## 返回值说明

| Field | Type | Description |
| --- | --- | --- |
| data.total | Long | 匹配过滤条件的订单总数 |
| data.totalBetScore | BigDecimal / null | 总下注金额；`hasStat=1` 且 `currency` 非空时返回 |
| data.totalSettleScore | BigDecimal / null | 总结算金额；`hasStat=1` 且 `currency` 非空时返回 |
| data.totalValidScore | BigDecimal / null | 总有效下注金额；`hasStat=1` 且 `currency` 非空时返回 |
| data.totalUserCount | Long / null | 去重用户数；`hasStat=1` 且 `currency` 非空时返回 |
| data.list | Object[] | 当前页订单列表 |

金额字段返回 `game_order_collection` 中的原始金额，不在查询响应中按 A0 currency unit 做除法换算。
