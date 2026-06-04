# A0 游戏服务配置接口

**Last Updated:** 2026-06-04

该接口用于维护每个 A0 的游戏服务配置。配置按 `kolUserId + configType` 唯一，写入接口为 upsert。

## 鉴权

所有 `/oapi/**` 请求都需要在请求头中携带 `token`。

| Header | Required | Description |
| --- | --- | --- |
| token | Yes | JWT token |
| kolUserId | No | 当传入该请求头时，系统优先使用该 A0 的 `TOKEN` 配置 `configKey` 校验 JWT；未传入或未配置时使用默认 OAPI token key。 |

## 配置类型

| configType | UI 名称 | 用途 |
| --- | --- | --- |
| BALANCE | 钱包配置 | 余额、扣分、优惠券、生命周期等平台调用 |
| TOKEN | token 配置 | 用户 token 校验和带 `kolUserId` 请求头的 OAPI 入站鉴权 |
| EVENT | event 配置 | 游戏事件服务配置 |

## 数据结构

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| kolUserId | Long | Yes | A0 用户 ID |
| configType | String | Yes | `BALANCE` / `TOKEN` / `EVENT` |
| configKey | String | No | JWT config key |
| url | String | No | 服务 URL |
| service | String | No | 服务名 |

```json
{
  "kolUserId": 11021170,
  "configType": "BALANCE",
  "configKey": "dWc2EpVYyJwKWeqF7YawteNup5Ai0Wx0",
  "url": "https://test-api.playpop.cc/gow/game/tokyo/callback",
  "service": "playpop"
}
```

## 新增或更新

- **Method:** `POST`
- **Path:** `/oapi/game-a0-service-config/create`

Body 同数据结构。按 `kolUserId + configType` upsert。

**Response Body**

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success"
}
```

## 更新

- **Method:** `POST`
- **Path:** `/oapi/game-a0-service-config/update`

行为同 `create`，按 `kolUserId + configType` upsert。

## 查询单条

- **Method:** `GET`
- **Path:** `/oapi/game-a0-service-config/get?kolUserId=11021170&configType=BALANCE`

**Response Body**

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success",
  "data": {
    "kolUserId": 11021170,
    "configType": "BALANCE",
    "configKey": "dWc2EpVYyJwKWeqF7YawteNup5Ai0Wx0",
    "url": "https://test-api.playpop.cc/gow/game/tokyo/callback",
    "service": "playpop"
  }
}
```

未查询到数据时 `data` 为 `null`。

## 查询 A0 全部配置

- **Method:** `GET`
- **Path:** `/oapi/game-a0-service-config/list/{kolUserId}`

**Response Body**

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success",
  "data": [
    {
      "kolUserId": 11021170,
      "configType": "BALANCE",
      "configKey": "dWc2EpVYyJwKWeqF7YawteNup5Ai0Wx0",
      "url": "https://test-api.playpop.cc/gow/game/tokyo/callback",
      "service": "playpop"
    },
    {
      "kolUserId": 11021170,
      "configType": "TOKEN",
      "configKey": "dWc2EpVYyJwKWeqF7YawteNup5Ai0Wx0",
      "url": "https://test-api.playpop.cc/gow/game/tokyo/callback",
      "service": "playpop"
    }
  ]
}
```

## 删除

- **Method:** `GET`
- **Path:** `/oapi/game-a0-service-config/delete?kolUserId=11021170&configType=BALANCE`

**Response Body**

```json
{
  "success": true,
  "errCode": "0",
  "errMessage": "success"
}
```
