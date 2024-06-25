# 使用事件

Sui 网络在链上存储无数对象，Move 代码可以使用这些对象执行操作。通常需要跟踪此活动，例如，发现模块铸造 NFT 的次数或统计智能合约生成的交易中的 SUI 数量。

为了支持活动监控，Move 提供了一个在 Sui 网络上发出事件的结构。当您与 Sui 网络建立连接时，您将在客户端和 Sui 网络节点之间创建双向交互式通信会话。通过开放会话，您可以订阅 Sui 网络添加到流中的特定事件，以创建事件的实时监控。

### Move 事件结构​

Sui 中的事件对象由以下属性组成：

- `id` ：包含交易摘要 ID 和事件序列的 JSON 对象。
- `packageId` ：发出事件的包的对象 ID。
- `transactionModule` ：执行事务的模块。
- `sender` ：触发事件的 Sui 网络地址。
- `type` ：发出的事件类型。
- `parsedJson` ：描述事件的 JSON 对象。
- `bcs` ：二进制规范序列化值。
- `timestampMs` ：Unix 纪元时间戳（以毫秒为单位）。

### 订阅事件​

如果你想订阅链上的事件，你首先需要知道有哪些事件可用。您通常知道或可以发现您自己的代码发出的事件，但当您需要从您不拥有的包订阅链上事件时，事情就没那么简单了。 Sui RPC 提供了 queryEvents 方法来查询链上包并返回您可以订阅的可用事件。

###  过滤事件​

您可以过滤代码目标的事件以进行查询或订阅。两个过滤器选项相似，但也有一些差异。

### 在 Move 中发出事件​

要在 Move 模块中创建事件，请添加 sui::event 依赖项。

```
use sui::event;
```

添加依赖项后，只要您要监视的操作触发，您就可以使用 `emit` 函数来触发事件。例如，以下代码是使用数字甜甜圈的示例应用程序的一部分。 `collect_profits` 函数处理 SUI 的集合，并在调用该函数时发出事件。要对该事件采取行动，您需要订阅它。

```
/// Take coin from `DonutShop` and transfer it to tx sender.
/// Requires authorization with `ShopOwnerCap`.
public fun collect_profits( _: &ShopOwnerCap, shop: &mut DonutShop, ctx: &mut TxContext ) {
    let amount = balance::value(&shop.balance);
    let profits = coin::take(&mut shop.balance, amount, ctx);
    // simply create new type instance and emit it.
    event::emit(ProfitsCollected { amount });
    transfer::public_transfer(profits, tx_context::sender(ctx));
}
```

### 订阅 Move 中的事件​

触发事件在很多时候并不够，您还需要能够侦听这些事件，以便可以对它们采取行动。在 Sui 中，您订阅这些事件并提供在事件触发时触发的逻辑。

Sui Full 节点支持使用通过 WebSocket API 传输的 JSON-RPC 通知的订阅功能。您可以直接与 RPC 交互（ suix_subscribeEvent、suix_subscribeTransaction），也可以使用 SDK（例如 Sui TypeScript SDK）。以下摘录自示例之一，使用 TypeScript SDK 创建对过滤器中标识的过滤器的异步订阅。

```
let unsubscribe = await provider.subscribeEvent({
    filter: { <PACKAGE_ID> },
    onMessage: (event) => {
        console.log("subscribeEvent", JSON.stringify(event, null, 2))
    }
});
```

Move智能合约可以调用其他发出事件的智能合约。例如， `Deepbook_utils` 可以调用 `dee9` 智能合约并发出此事件。请注意，使用包、事务模块查询 `dee9/clob_v2` 会错过以下事件，即使它实际上是 `dee9` 包发出的事件。此问题当前的解决方法是了解您关心的所有 `packageId` 并在 `queryEvent` 调用中搜索它们。

```
{
	"id": {
		"txDigest": "bZnc1E7k1fJYLxWihfre5xCw1tX1CyAN6579zypJeiU",
		"eventSeq": "0"
	},
	"packageId": "0x158f2027f60c89bb91526d9bf08831d27f5a0fcb0f74e6698b9f0e1fb2be5d05",
	"transactionModule": "deepbook_utils",
	"sender": "0x4419ae182ac112bb065bda2146136ed02524ee2611478bfe8ca5d3835bee4af6",
	"type": "0xdee9::clob_v2::OrderPlaced<0x2::sui::SUI, 0x5d4b302506645c37ff133b98c4b50a5ae14841659738d6d733d59d0d217a93bf::coin::COIN>",
	"parsedJson": {
		"base_asset_quantity_placed": "1000000000",
		"client_order_id": "20082022",
		"expire_timestamp": "1697121171540",
		"is_bid": false,
		"order_id": "9223372036854945121",
		"original_quantity": "1000000000",
		"owner": "0x8c23e5e23c6eb654d69f8ae7de3be23584f435cad81fa4b9cb024b6c989b7818",
		"pool_id": "0x7f526b1263c4b91b43c9e646419b5696f424de28dda3c1e6658cc0a54558baa7",
		"price": "500000"
	},
	"bcs": "2pWctGGQ9KULfmnzNtGuPpggLQrj1ZiUQaxva4neM6QWAtUAkuPAzU2eGrdZaGHti3bsUefDioUwwYoVR3bYBkG7Gxf5JVVSxxqTqzxdg5os5ESwFaP69ZcrNsya4G9rHK4KBac9i3m1MseN38xDwMvAMx3"
}
```

```
{
	"id": {
		"txDigest": "896CKHod5GQ4kzhF7EwTAGyhQBdaTb9rQS41dcL76gj8",
		"eventSeq": "0"
	},
	"packageId": "0x000000000000000000000000000000000000000000000000000000000000dee9",
	"transactionModule": "clob_v2",
	"sender": "0xf821d3483fc7725ebafaa5a3d12373d49901bdfce1484f219daa7066a30df77d",
	"type": "0xdee9::clob_v2::OrderPlaced<0xbc3a676894871284b3ccfb2eec66f428612000e2a6e6d23f592ce8833c27c973::coin::COIN, 0x5d4b302506645c37ff133b98c4b50a5ae14841659738d6d733d59d0d217a93bf::coin::COIN>",
	"parsedJson": {
		"base_asset_quantity_placed": "5000000",
		"client_order_id": "1696545636947311087",
		"expire_timestamp": "1696549236947",
		"is_bid": true,
		"order_id": "562414",
		"original_quantity": "5000000",
		"owner": "0xf995d6df20e18421928ff0648bd583ccdf384ab05791d8be21d32977a37dacfc",
		"pool_id": "0xf0f663cf87f1eb124da2fc9be813e0ce262146f3df60bc2052d738eb41a25899",
		"price": "274518000000"
	},
	"bcs": "4SgemkCzrqEsTHLFgMcbUtttZCf2CrEH2njjFL1rizCHzvAoYsToGrbFLffQPtGxsSt96Xr4j2SLNeLcBGKeYXDrVYWqivhf3551Mqj71DZBxq5D1Qwfgh1TKeF43Jz4b4XH1nEpkya2Pr8515vzJbHUkpP"
}
```

###  示例​

此示例利用 Sui TypeScript SDK 订阅 ID 为 `<PACKAGE_ID>` 的包发出的事件。每次事件触发时，代码都会向控制台显示响应。

### TypeScript

要创建事件订阅，您可以使用基本的 Node.js 应用程序。您需要 Sui TypeScript SDK，因此请在项目的根目录下使用 `npm install @mysten/sui` 安装该模块。在您的 TypeScript 代码中，导入 `JsonRpcProvider` 和来自库的连接。

```
import { getFullnodeUrl, SuiClient } from '@mysten/sui/client';

// Package is on Testnet.
const client = new SuiClient({
	url: getFullnodeUrl('testnet'),
});
const Package = '<PACKAGE_ID>';

const MoveEventType = '<PACKAGE_ID>::<MODULE_NAME>::<METHOD_NAME>';

console.log(
	await client.getObject({
		id: Package,
		options: { showPreviousTransaction: true },
	}),
);

let unsubscribe = await client.subscribeEvent({
	filter: { Package },
	onMessage: (event) => {
		console.log('subscribeEvent', JSON.stringify(event, null, 2));
	},
});

process.on('SIGINT', async () => {
	console.log('Interrupted...');
	if (unsubscribe) {
		await unsubscribe();
		unsubscribe = undefined;
	}
});
```

### 响应

当订阅的事件触发时，该示例将显示事件的以下 JSON 表示形式。

```
subscribeEvent {
  "id": {
    "txDigest": "HkCBeBLQbpKBYXmuQeTM98zprUqaACRkjKmmtvC6MiP1",
    "eventSeq": "0"
  },
  "packageId": "0x2d6733a32e957430324196dc5d786d7c839f3c7bbfd92b83c469448b988413b1",
  "transactionModule": "coin_flip",
  "sender": "0x46f184f2d68007e4344fffe603c4ccacd22f4f28c47f321826e83619dede558e",
  "type": "0x2d6733a32e957430324196dc5d786d7c839f3c7bbfd92b83c469448b988413b1::coin_flip::Outcome",
  "parsedJson": {
    "bet_amount": "4000000000",
    "game_id": "0xa7e1fb3c18a88d048b75532de219645410705fa48bfb8b13e8dbdbb7f4b9bbce",
    "guess": 0,
    "player_won": true
  },
  "bcs": "3oWWjWKRVu115bnnZphyDcJ8EyF9X4pgVguwhEtcsVpBf74B6RywQupm2X",
  "timestampMs": "1687912116638"
}
```

###  Rust SDK​

```
use futures::StreamExt;
use sui_sdk::rpc_types::EventFilter;
use sui_sdk::SuiClientBuilder;
use anyhow::Result;

#[tokio::main]
async fn main() -> Result<()> {
    let sui = SuiClientBuilder::default()
        .ws_url("wss://fullnode.mainnet.sui.io:443")
        .build("https://fullnode.mainnet.sui.io:443")
        .await.unwrap();
    let mut subscribe_all = sui.event_api().subscribe_event(EventFilter::All(vec![])).await?;
    loop {
        println!("{:?}", subscribe_all.next().await);
    }
}
```

### 过滤事件查询​

要过滤从查询返回的事件，请使用以下数据结构。

:::info
这组过滤器仅适用于事件查询API。它与为订阅 API 提供的过滤器不同（请参阅以下部分）。特别是，它不支持 `"All": [...]` 、 `"Any": [...]` 、 `"And": [_, _]` 、 `"Or": [_, _]` 和 `"Not": _ `等组合。
:::

| 查询 | 描述 | JSON-RPC 参数示例 |
|-----|-----|-------------------|
| All | 所有事件 | `{"All"}`              |
| Transaction | 从指定事务发出的事件 | `{"Transaction":"DGUe2TXiJdN3FI6MH1FwghYbiHw+NKu8Nh579zdFtUk="}`     |
| MoveModule | 从指定的 Move 模块发出的事件 | `{"MoveModule":{"package":"<PACKAGE-ID>", "module":"nft"}}`   |
| MoveEventModule	 | 发出的事件，在指定的 Move 模块上定义。 | `{"MoveEventModule": {"package": "<DEFINING-PACKAGE-ID>", "module": "nft"}}` |
| MoveEvent | Move 事件的结构名称 | `{"MoveEvent":"::nft::MintNFTEvent"}`   |
| EventType | 事件部分描述的事件类型 | `{"EventType": "NewObject"}`              |
| Sender | 按发件人地址查询 | `{"Sender":"0x008e9c621f4fdb210b873aab59a1e5bf32ddb1d33ee85eb069b348c234465106"}`              |
| Recipient | 按收件人查询 | `{"Recipient":{"AddressOwner":"0xa3c00467938b392a12355397bdd3d319cea5c9b8f4fc9c51b46b8e15a807f030"}}`   |
| Object | 返回与给定对象关联的事件 | `{"Object":"0x727b37454ab13d5c1dbb22e8741bff72b145d1e660f71b275c01f24e7860e5e5"}`              |
| TimeRange | 返回在 [start_time, end_time] 间隔内发出的事件 | `{"TimeRange":{"startTime":1669039504014, "endTime":1669039604014}}`   |

### 过滤订阅事件​

要创建订阅，您可以设置过滤器以仅返回您有兴趣侦听的事件集。

:::info
这组过滤器仅适用于事件订阅 API。它与为查询 API 提供的过滤器不同（请参阅上一节）。特别是，它支持 `"All": [...]` 、 `"Any": [...]` 、 `"And": [_, _]` 、 `"Or": [_, _]` 和 `"Not": _` 等组合。
:::

| 过滤 | 描述 | JSON-RPC 参数示例 |
|-----|-----|-------------------|
| Package | Move 包 ID | `{"Package":"<PACKAGE-ID>"}`              |
| MoveModule | 从指定的 Move 模块发出的事件 | `{"MoveModule":{"package":"<PACKAGE-ID>", "module":"nft"}}`   |
| MoveEventType | Move 代码中定义的 Move 事件类型 | `{"MoveEventType":"<PACKAGE-ID>::nft::MintNFTEvent"}`     |
| MoveEventModule	 | 发出的事件，在指定的 Move 模块上定义。 | `{"MoveEventModule": {"package": "<DEFINING-PACKAGE-ID>", "module": "nft"}}` |
| MoveEventField | 使用 Move 事件对象中的数据字段进行过滤 | `{"MoveEventField":{ "path":"/name", "value":"NFT"}}`   |
| SenderAddress | 开始交易的地址 | `{"SenderAddress":"0x008e9c621f4fdb210b873aab59a1e5bf32ddb1d33ee85eb069b348c234465106"}`              |
| Sender | 发件人地址 | `{"Sender":"0x008e9c621f4fdb210b873aab59a1e5bf32ddb1d33ee85eb069b348c234465106"}`              |
| Transaction | 交易哈希 | `{"Transaction":"ENmjG42TE4GyqYb1fGNwJe7oxBbbXWCdNfRiQhCNLBJQ"}`   |
| TimeRange | 时间范围（以毫秒为单位） | `{"TimeRange":{"startTime":1669039504014, "endTime":1669039604014}}`   |
