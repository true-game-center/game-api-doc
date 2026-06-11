# A0 Currency Bet/Settle Statistics API

**Last Updated:** 2026-06-11

This interface returns the total bet amount and payout amount from `game_order_collection` for one A0 and one currency.

## Endpoint

- **Method:** `POST`
- **Path:** `/oapi/game-order-collection/a0-currency-bet-settle-stat`
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
  "currency": "USD",
  "beginTime": 1780488000000,
  "endTime": 1780574399000
}
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| kolUserId | Long | Yes | A0 user ID, mapped to `game_order_collection.kol_user_id`. Alias `a0` is also accepted. |
| currency | String | Yes | Order currency, mapped to `game_order_collection.currency`. |
| beginTime | Long | Yes | Start time as a millisecond timestamp. The filter uses `game_order_collection.bet_time >= beginTime`. |
| endTime | Long | Yes | End time as a millisecond timestamp. The filter uses `game_order_collection.bet_time <= endTime`. |

## Calculation Rules

| Field | Rule |
| --- | --- |
| Time range | Filter by `game_order_collection.bet_time`, inclusive of both `beginTime` and `endTime` |
| betScore | `sum(coalesce(bet_score, 0))` for rows where `kol_user_id = kolUserId`, `currency = currency`, and `bet_time` is within the requested range |
| settleScore | `sum(coalesce(settle_score, 0))` for rows where `kol_user_id = kolUserId`, `currency = currency`, and `bet_time` is within the requested range |

The response returns raw amounts in the requested order currency. It does not apply A0 currency-unit conversion.

## Response Body

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success",
  "data": {
    "kolUserId": 11012184,
    "currency": "USD",
    "betScore": 1000.00000000,
    "settleScore": 760.00000000
  }
}
```

When no matching orders exist, `betScore` and `settleScore` return `0`.
