# getAllBalances

Return the total coin balance for all coin type, owned by the address owner.

## 参数

| 参数          | 是否必须 | 描述                       |
|----------------|----|-------------------------------|
| owner&lt;address&gt; | Y  | Owner的地址                    |

## 返回值

```rust
Vec<Balance><[Balance]>
```

## 示例

获取一个地址的所有余额。

### 请求

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "bfc_getAllBalances",
  "params": [
    "0x94f1a597b4e8f709a396f7f6b1482bdcd65a673d111e49286c527fab7c2d0961"
  ]
}
```

### 响应

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": [
    {
      "coinType": "0x2::bfc::BFC",
      "coinObjectCount": 15,
      "totalBalance": "3000000000",
      "lockedBalance": {}
    }
  ]
}
```
