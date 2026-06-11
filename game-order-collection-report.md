# Game Order Collection Report API

**Last Updated:** 2026-06-11

This interface returns multidimensional game order report metrics from `game_order_collection`.

## Endpoint

- **Method:** `POST`
- **Path:** `/oapi/game-order-collection/report`
- **Content-Type:** `application/json`

## Authentication

All `/oapi/**` requests must include a `token` header.

| Header | Required | Description |
| --- | --- | --- |
| token | Yes | JWT token |

## Request Body

```json
{
  "kolUserId": 11012184,
  "userId": 11011234,
  "reportDate": "2026-06-11",
  "beginDate": "2026-06-01",
  "endDate": "2026-06-11",
  "gameSupplierId": 1486042272511296851,
  "gameCategoryId": 1491818780504883931,
  "gameId": 1497254068404215812,
  "currency": "USD"
}
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| kolUserId | Long | Yes | A0 user ID |
| userId | Long | No | End user ID |
| reportDate | Date | No | Single report date in `yyyy-MM-dd`. When present, it overrides `beginDate` and `endDate`. |
| beginDate | Date | No | Start date in `yyyy-MM-dd`, inclusive |
| endDate | Date | No | End date in `yyyy-MM-dd`, inclusive |
| gameSupplierId | Long | No | Internal game supplier ID, maps to `supplier_id` |
| gameCategoryId | Long | No | Internal game category ID, maps to `category_id` |
| gameId | Long | No | Internal game ID, maps to `game_id` |
| currency | String | No | Currency filter. When present, SQL calculations only use rows where `game_order_collection.currency = currency`; when omitted, all currencies are included. |

## Calculation Rules

| Metric | Rule |
| --- | --- |
| totalOrderCount | Count of matching orders |
| gameRoundCount | Same value as `totalOrderCount` |
| User buckets | Count users by their matching order count: `<=10`, `>10 and <=50`, `>50 and <=200`, `>200 and <=1000`, `>1000` |
| totalBetScore | `sum(bet_score)` grouped by currency, converted by the A0 currency unit, then summed |
| totalValidBetScore | `sum(valid_bet_score)` grouped by currency, converted by the A0 currency unit, then summed |
| totalDrawScore | `sum(bet_score)` for `order_status = 2`, grouped by currency, converted by the A0 currency unit, then summed |
| totalSettleScore | `sum(settle_score)` grouped by currency, converted by the A0 currency unit, then summed |
| totalWinLoss | `totalBetScore - totalSettleScore` |
| returnRate | `totalSettleScore / totalBetScore`; returns `0` when `totalBetScore` is `0` |

The optional `currency` filter applies to both the user bucket SQL and amount aggregation SQL.

## Response Body

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success",
  "data": {
    "totalOrderCount": 128,
    "gameRoundCount": 128,
    "trafficUserCount": 10,
    "lightUserCount": 3,
    "mediumUserCount": 1,
    "heavyUserCount": 0,
    "whaleUserCount": 0,
    "totalBetScore": 1000.00000000,
    "totalValidBetScore": 900.00000000,
    "totalDrawScore": 20.00000000,
    "totalSettleScore": 760.00000000,
    "totalWinLoss": 240.00000000,
    "returnRate": 0.760000
  }
}
```
