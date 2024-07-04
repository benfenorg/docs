# getAllCoins

Return all Coin objects owned by an address.

## 参数

| 参数            | 是否必须 | 描述                       |
|--------------------|----|-------------------------------|
| owner&lt;address&gt;     | Y  | Owner的地址                    |
| cursor&lt;ObjectID&gt;   | N  | Paging cursor                 |
| limit&lt;uint&gt;        | N  | 每页返回的最大数量               |

## 返回值

```rust
CoinPage< Page_for_Coin_and_ObjectID >
```

## 示例

Gets all coins for the address in the request body. Begin listing the coins that are after the provided cursor value and return only the limit amount of results per page.

### 请求

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "sbf_getAllCoins",
  "params": [
    "0x41f5975e3c6bd5c95f041a8493ad7e9934be26e69152d2c2e86d8a9bdbd242b3",
    "0x2564cd31a71cf9833609b111436d8f0f47b7f8b9927ec3f8975a1dcbf9b25564",
    3
  ]
}
```

### 响应

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "data": [
      {
        "coinType": "0x2::bfc::BFC",
        "coinObjectId": "0x91825debff541cf4e08b5c5f7296ff9840e6f0b185af93984cde8cf3870302c0",
        "version": "103626",
        "digest": "7dp5WtTmtGp83EXYYFMzjBJRFeSgR67AzqMETLrfgeFx",
        "balance": "200000000",
        "previousTransaction": "9WfFUVhjbbh4tWkyUse1QxzbKX952cyXScH7xJNPB2vQ"
      },
      {
        "coinType": "0x2::bfc::BFC",
        "coinObjectId": "0x48a53f22e2e901ea2a5bf44fdd5bb94a1d83b6efc4dd779f0890ca3b1f6ba997",
        "version": "103626",
        "digest": "9xLdMXezY8d1yRA2TtN6pYjapyy2EVKHWNriGPFGCFvd",
        "balance": "200000000",
        "previousTransaction": "Byq9SyV7x6fvzaf88YRA9JM8vLbVLJAqUX8pESDmKcgw"
      },
      {
        "coinType": "0x2::bfc::BFC",
        "coinObjectId": "0x6867fcc63161269c5c0c73b02229486bbaff319209dfb8299ced3b8609037997",
        "version": "103626",
        "digest": "5xexWFq6QpGHBQyC9P2cbAJXq9qm2EjzfuRM9NwS1uyG",
        "balance": "200000000",
        "previousTransaction": "CEjwHmo98nAiYhSMfKoSDvUMtfKJ6ge6Uj4wKotK4MPZ"
      }
    ],
    "nextCursor": "0x861c5e055605b2bb1199faf653a8771e448930bc95a0369fad43a9870a2e5878",
    "hasNextPage": true
  }
}
```
