# Game Supplier Management API

**Last Updated:** 2026-06-09

This document covers the Game Center supplier management fields that affect order collection and supplier wallet routing.

## Supplier Wallet Fields

`game_supplier` supports the following wallet routing fields.

| Field | Type | Description |
|-------|------|-------------|
| `enableSupplierWallet` | integer | Whether to enable supplier wallet routing. `1` = enabled, `0` = disabled. Default: `0`. |
| `supplierWalletUrl` | string | Supplier wallet base URL. Optional. Used only when `enableSupplierWallet = 1` and the value is not blank. |

Existing order collection fields remain unchanged.

| Field | Type | Description |
|-------|------|-------------|
| `needCollectOrder` | integer | Whether to collect supplier orders. `1` = collect, `0` = do not collect. |
| `gameProxyUrl` | string | Game order collection URL / supplier game proxy URL. |

## Affected APIs

The following supplier APIs accept and return the wallet fields above:

| Method | Path | Notes |
|--------|------|-------|
| `POST` | `/oapi/game-supplier/create` | Creates a supplier. |
| `POST` | `/oapi/game-supplier/update` | Updates a supplier. |
| `GET` | `/oapi/game-supplier/{id}` | Returns supplier detail. |
| `GET` | `/oapi/game-supplier/{id}/withI18n` | Returns supplier detail with i18n names. |
| `POST` | `/oapi/game-supplier/page` | Returns supplier page data. `enableSupplierWallet` can be used as a filter. |

## Create / Update Example

```json
{
  "gameSupplierCode": "jili",
  "gameSupplierName": "JILI",
  "platformSupplierId": "jili",
  "thirdSupplierId": "jili",
  "imgIconId": "jili",
  "sortNo": 1,
  "state": 1,
  "openType": 0,
  "needCollectOrder": 1,
  "gameProxyUrl": "https://game-proxy-jili.example.com",
  "enableSupplierWallet": 1,
  "supplierWalletUrl": "https://wallet-jili.example.com"
}
```

## Score Cost Routing

Game Center's downstream score-cost Feign path remains fixed:

```text
POST /oapi/score/cost
```

When a score change reaches Game Center and the local supplier can be resolved:

1. If `enableSupplierWallet = 1` and `supplierWalletUrl` is not blank, Game Center uses `supplierWalletUrl` as the Feign base URI and calls `{supplierWalletUrl}/oapi/score/cost`.
2. Otherwise, Game Center keeps the existing A0 service routing: it resolves `game_a0_service_config` by `kolUserId + BALANCE`, then falls back to the default `game-core.oapi.token.url` when no A0 config exists.

The JWT token generation still follows the existing A0 `BALANCE` configuration.
