# 多重签名认证

以下步骤演示了如何创建多重签名交易，然后使用 Bfc CLI 将其提交到本地网络。交易可以是对象的转移、包的发布或升级、SUI的支付等。要了解如何设置本地网络，请参阅连接到本地网络。

要了解有关如何使用 TypeScript SDK 创建多重签名地址和创建多重签名交易的更多信息，请参阅 SDK 文档以了解详细信息。

## 第1步：创建密钥​

使用以下命令为每个支持的密钥方案生成 Bfc 地址和密钥并将其添加到 `bfc.keystore` ，然后列出密钥。

使用 `bfc client` 创建不同密钥方案的BFC地址。

```plain
bfc client new-address ed25519
bfc client new-address secp256k1
bfc client new-address secp256r1
```

## 第2步：将密钥添加到 Bfc 密钥库​

使用 bfc keytool 列出您在上一步中创建的签名。

```plain
bfc keytool list
```

响应类似于以下内容，但显示实际地址、密钥和对等 ID：

```plain
╭────────────────────────────────────────────────────────────────────────────────────────────╮
│ ╭─────────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ suiAddress      │  <BFC-ADDRESS>                                                       │ │
│ │ publicBase64Key │  <PUBLIC-KEY>                                                        │ │
│ │ keyScheme       │  ed25519                                                             │ │
│ │ flag            │  0                                                                   │ │
│ │ peerId          │  <PEER-ID>                                                           │ │
│ ╰─────────────────┴──────────────────────────────────────────────────────────────────────╯ │
│ ╭─────────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ suiAddress      │  <BFC-ADDRESS>                                                       │ │
│ │ publicBase64Key │  <PUBLIC-KEY>                                                        │ │
│ │ keyScheme       │  secp256k1                                                           │ │
│ │ flag            │  0                                                                   │ │
│ │ peerId          │  <PEER-ID>                                                           │ │
│ ╰─────────────────┴──────────────────────────────────────────────────────────────────────╯ │
│ ╭─────────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ suiAddress      │  <BFC-ADDRESS>                                                       │ │
│ │ publicBase64Key │  <PUBLIC-KEY>                                                        │ │
│ │ keyScheme       │  secp256r1                                                           │ │
│ │ flag            │  0                                                                   │ │
│ │ peerId          │  <PEER-ID>                                                           │ │
│ ╰─────────────────┴──────────────────────────────────────────────────────────────────────╯ │
╰────────────────────────────────────────────────────────────────────────────────────────────╯
```

## 第3步：创建多重签名地址​

要创建多重签名地址，请输入用于多重签名地址的公钥列表以及相应权重和阈值的列表（将 `<VARIABLES>` 替换为实际值）。

```plain
bfc keytool multi-sig-address --pks <PUBLIC-KEY-ED25519> <PUBLIC-KEY-SECPK1> <PUBLIC-KEY-SECP256R1> --weights 1 2 3 --threshold 3
```

响应类似于以下内容：

```plain
╭─────────────────┬────────────────────────────────────────────────────────────────────────────────────╮
│ multisigAddress │  <MULTISIG-ADDRESS>                                                                │
│ multisig        │ ╭────────────────────────────────────────────────────────────────────────────────╮ │
│                 │ │ ╭─────────────────┬──────────────────────────────────────────────────────────╮ │ │
│                 │ │ │ address         │  <BFC-ADDRESS>                                           │ │ │
│                 │ │ │ publicBase64Key │  <PUBLIC-KEY>                                            │ │ │
│                 │ │ │ weight          │  1                                                       │ │ │
│                 │ │ ╰─────────────────┴──────────────────────────────────────────────────────────╯ │ │
│                 │ │ ╭─────────────────┬──────────────────────────────────────────────────────────╮ │ │
│                 │ │ │ address         │  <BFC-ADDRESS>                                           │ │ │
│                 │ │ │ publicBase64Key │  <PUBLIC-KEY>                                            │ │ │
│                 │ │ │ weight          │  2                                                       │ │ │
│                 │ │ ╰─────────────────┴──────────────────────────────────────────────────────────╯ │ │
│                 │ │ ╭─────────────────┬──────────────────────────────────────────────────────────╮ │ │
│                 │ │ │ address         │  <BFC-ADDRESS>                                           │ │ │
│                 │ │ │ publicBase64Key │  <PUBLIC-KEY>                                            │ │ │
│                 │ │ │ weight          │  3                                                       │ │ │
│                 │ │ ╰─────────────────┴──────────────────────────────────────────────────────────╯ │ │
│                 │ ╰────────────────────────────────────────────────────────────────────────────────╯ │
╰─────────────────┴────────────────────────────────────────────────────────────────────────────────────╯
```

## 第4步：将对象发送到多重签名地址​

此示例按照连接到本地网络中的指导，使用默认 URL 从本地网络请求 Gas。如果继续操作，请务必将 `<MULTISIG-ADDR>` 替换为您在上一步中收到的地址。

```plain
curl --location --request POST 'http://127.0.0.1:9123/gas' --header 'Content-Type: application/json' --data-raw "{ \"FixedAmountRequest\": { \"recipient\": \"<MULTISIG-ADDR>\" } }"
```

响应类似于以下内容：

```plain
{"transferred_gas_objects":[{"amount":200000,"id":"<OBJECT-ID>", ...}]}
```

## 第5步：序列化交易​

本节演示如何使用属于多重签名地址的对象并序列化要签名的传输。 `tx_bytes` 值可以是任何序列化交易数据，其中发送者是多重签名地址。对 `bfc client -h` 中支持的命令使用 `--serialize-unsigned-transaction` 标志（ `publish` 、 `upgrade` 、 `call` 、 `transfer` 、 `pay` 、 `pay-all-bfc` 、 `pay-bfc` 、 `split` 、 `merge-coin` ) 输出 Base64 编码的交易字节。

```plain
bfc client transfer --to <BFC-ADDRESS> --object-id <OBJECT-ID> --serialize-unsigned-transaction

Raw tx_bytes to execute: <TX_BYTES>
```

## 第6步：用两个密钥签署交易​

使用以下代码示例使用 bfc.keystore 中的两个密钥对交易进行签名。您可以使用其他工具来执行此操作，只要将其序列化为 `flag || sig || pk` 即可。

```plain
bfc keytool sign --address <BFC-ADDRESS> --data <TX_BYTES>

Raw tx_bytes to execute: <TX_BYTES>
Serialized signature (`flag || sig || pk` in Base64): $SIG_1

bfc keytool sign --address <BFC-ADDRESS> --data <TX_BYTES>

Raw tx_bytes to execute: <TX_BYTES>
Serialized signature (`flag || sig || pk` in Base64): $SIG_2
```

## 第7步：将各个签名组合成多重签名​

此示例演示了如何组合两个签名：

```plain
bfc keytool multi-sig-combine-partial-sig --pks <PUBLIC-KEY-1> <PUBLIC-KEY-2> <PUBLIC-KEY-3> --weights 1 2 3 --threshold 3 --sigs <SIGNATURE-1> <SIGNATURE-2>

multisig address: <MULTISIG-ADDRESS> # Informational
multisig parsed: <HUMAN-READABLE-STRUCT> # Informational
multisig serialized: <SERIALIZED-MULTISIG>
```

您只需要权重总和为 >=k 的参与签名者的签名。您必须提供所有公钥及其权重，以及定义多重签名地址的阈值。

## 第8步：使用多重签名执行交易​

使用 bfc client 使用多重签名执行交易：

```plain
bfc client execute-signed-tx --tx-bytes <TX_BYTES> --signatures <SERIALIZED-MULTISIG>
```
