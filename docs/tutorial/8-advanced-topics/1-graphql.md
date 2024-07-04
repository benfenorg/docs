# 迁移到 GraphQL

本指南将 JSON-RPC 查询与其等效的 GraphQL 查询进行比较。虽然可以使用本指南系统地将 JSON-RPC 查询（例如 sui_getTotalTransactionBlocks ）重写为其 GraphQL 对应项，但建议您重新访问应用程序的查询模式，以充分利用 GraphQL 的灵活性提供涉及多个潜在嵌套端点（例如交易、余额、硬币）的查询服务，并使用以下示例来了解这两个 API 如何表达相似的概念。

有关所有可用 GraphQL 功能的完整列表，请参阅参考资料。

### 示例1：获取总交易区块​

目标是获取网络中交易块的总数。

JSON-RPC：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "sui_getTotalTransactionBlocks",
  "params": []
}
```

GraphQL：

```graph
query {
  checkpoint {
    networkTotalTransactions
  }
}
```

### 示例2：获取特定交易区块​

目标是通过摘要获取交易块。

JSON-RPC：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "sui_getTransactionBlock",
  "params": [
    "Hay2tj3GcDYcE3AMHrej5WDsHGPVAYsegcubixLUvXUF",
    {
      "showInput": true,
      "showRawInput": false,
      "showEffects": true,
      "showEvents": true,
      "showObjectChanges": false,
      "showBalanceChanges": false
    }
  ]
}
```

GraphQL：

```graph
query {
  transactionBlock(digest: "Hay2tj3GcDYcE3AMHrej5WDsHGPVAYsegcubixLUvXUF") {
    gasInput {
      gasSponsor {
        address
      }
      gasPrice
      gasBudget
    }
    effects {
      status
      timestamp
      checkpoint {
        sequenceNumber
      }
      epoch {
        epochId
        referenceGasPrice
      }
    }
  }
}
```

### 示例3：获取某个地址拥有的Coin对象​

目标是返回地址拥有的所有 `Coin<0x2::sui::SUI>` 对象。

JSON-RPC：

```json
query {
  "jsonrpc": "2.0",
  "id": 1,
  "method": "suix_getCoins",
  "params": [
    "0x5094652429957619e6efa79a404a6714d1126e63f551f4b6c7fb76440f8118c9", //owner
    "0x2::sui::SUI",                                                      //coin type
    "0xe5c651321915b06c81838c2e370109b554a448a78d3a56220f798398dde66eab", //cursor
    3 //limit
  ]
}
```

GraphQL：

```graph
query {
  address(address: "0x5094652429957619e6efa79a404a6714d1126e63f551f4b6c7fb76440f8118c9") {
    coins(
      first: 3,
      after: "IAB3ha2PEA4ESRF4UErsJufJEwYpmSbCq7UNpxIHnLhG",
      type: "0x2::sui::SUI"
    ) {
      nodes {
        address
      }
    }
  }
}
```
