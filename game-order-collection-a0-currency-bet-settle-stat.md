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
  "currency": "USD"
}
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| kolUserId | Long | Yes | A0 user ID, mapped to `game_order_collection.kol_user_id`. Alias `a0` is also accepted. |
| currency | String | Yes | Order currency, mapped to `game_order_collection.currency`. |

## Calculation Rules

| Field | Rule |
| --- | --- |
| betScore | `sum(coalesce(bet_score, 0))` for rows where `kol_user_id = kolUserId` and `currency = currency` |
| settleScore | `sum(coalesce(settle_score, 0))` for rows where `kol_user_id = kolUserId` and `currency = currency` |

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
