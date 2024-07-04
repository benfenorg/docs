# 受监管的 Coin 和拒绝名单

Bfc Coin标准提供了 `create_regulated_currency` 函数来创建硬币。此函数与 `create_currency` 不同，它会生成一种代币，您可以阻止某些地址在交易中使用这些代币。这种能力是稳定币等资产的要求。

在幕后， `create_regulated_currency` 使用 `create_currency` 函数创建代币，同时还生成一个 `DenyCap` 对象，允许其持有者控制对代币拒绝列表的访问在 `DenyList` 对象中。因此，使用 `create_regulated_currency` 创建硬币的方式与前面的示例类似，只是将 `DenyCap` 对象传输到模块发布者。

```
// examples/move/coin/sources/regcoin.move

module examples::regcoin {
    use bfc::coin::{Self, DenyCap};
    use bfc::deny_list::{DenyList};

    public struct REGCOIN has drop {}

    fun init(witness: REGCOIN, ctx: &mut TxContext) {
        let (treasury, deny_cap, metadata) = coin::create_regulated_currency(witness, 6, b"REGCOIN", b"", b"", option::none(), ctx);
        transfer::public_freeze_object(metadata);
        transfer::public_transfer(treasury, ctx.sender());
        transfer::public_transfer(deny_cap, ctx.sender())
    }
}
```

当您使用 `bfc client publish` 部署上一个模块时，控制台会响应事务效果，包括创建以下对象：

```
...

Object Changes

Created Objects:

   ObjectID: <OBJECT-ID>
   Sender: <SENDER-ADDR>
   Owner: Immutable
   ObjectType: 0x2::coin::CoinMetadata<<PACKAGE-ID>::regcoin::REGCOIN>
   Version: <VERSION-NUMBER>
   Digest: <DIGEST-HASH>

   ObjectID: <OBJECT-ID>
   Sender: <SENDER-ADDR>
   Owner: Account Address ( <PUBLISHER-ADDRESS )
   ObjectType: 0x2::package::UpgradeCap
   Version: <VERSION-NUMBER>
   Digest: <DIGEST-HASH>

   ObjectID: <OBJECT-ID>
   Sender: <SENDER-ADDR>
   Owner: Immutable
   ObjectType: 0x2::coin::RegulatedCoinMetadata<<PACKAGE-ID>::regcoin::REGCOIN>
   Version: <VERSION-NUMBER>
   Digest: <DIGEST-HASH>

   ObjectID: <OBJECT-ID>
   Sender: <SENDER-ADDR>
   Owner: Account Address ( <PUBLISHER-ADDRESS )
   ObjectType: 0x2::coin::DenyCap<<PACKAGE-ID>::regcoin::REGCOIN>
   Version: <VERSION-NUMBER>
   Digest: <DIGEST-HASH>


   ObjectID: <OBJECT-ID>
   Sender: <SENDER-ADDR>
   Owner: Account Address ( <PUBLISHER-ADDRESS )
   ObjectType: 0x2::coin::TreasuryCap<PACKAGE-ID>::regcoin::REGCOIN>
   Version: <VERSION-NUMBER>
   Digest: <DIGEST-HASH>

...
```

您可能已经注意到，发布操作会创建一个 `RegulatedCoinMetadata` 对象以及标准 `CoinMetadata` 对象。但是，您不需要在 `RegulatedCoinMetadata` 对象上显式调用 `freeze_object` ，因为 `create_regulated_currency` 会自动执行此操作。

输出还显示了发布者现在拥有的三个对象：用于包升级的 `UpgradeCap` 、用于铸造或销毁硬币的 `TreasuryCap` 以及用于添加或删除的 `DenyCap` 该硬币的拒绝名单的地址或来自该名单的地址。

###  拒绝名单​

Bfc 框架提供了一个 `DenyList` 单例共享对象， `DenyCap` 的持有者可以访问该对象来指定无法使用 Bfc 核心类型的地址列表。然而， `DenyList` 的初始用例侧重于限制对指定类型硬币的访问。例如，当在 Bfc 上创建受监管的代币时，需要能够阻止某些地址将其用作交易的输入，这非常有用。 Bfc 上受监管的代币满足任何要求能够防止已知不良行为者获取这些代币的法规。

####  操作拒绝名单​

为了能够操作分配给您的货币拒绝列表的地址，您必须在前面的示例中添加一些函数。

```
// examples/move/coin/sources/regcoin.move

public fun add_addr_from_deny_list(denylist: &mut DenyList, denycap: &mut DenyCap<REGCOIN>, denyaddy: address, ctx: &mut TxContext) {
    coin::deny_list_add(denylist, denycap, denyaddy, ctx);
}

public fun remove_addr_from_deny_list(denylist: &mut DenyList, denycap: &mut DenyCap<REGCOIN>, denyaddy: address, ctx: &mut TxContext){
    coin::deny_list_remove(denylist, denycap, denyaddy, ctx);
}
```

要使用这些函数，您需要传递 `DenyList` 对象 ( `0x403` )、 `DenyCap` 对象 ID 以及要添加或删除的地址。使用 Bfc CLI，您可以将 `bfc client call` 与所需信息一起使用：

```
bfc client call --function add_addr_from_deny_list --module regcoin --package <PACKAGE-ID> --args <DENY-LIST> <DENY-CAP> <ADDRESS-TO-DENY> --gas-budget <GAS-AMOUNT>
Transaction Digest: <DIGEST-HASH>
```

控制台显示来自网络的响应，您可以在其中验证 `DenyList` 对象是否已发生变化。

```
...

MutatedObjects:

  ObjectID: 0x0...403               
  Sender: <SENDER-ADDRESS>
  Owner: Shared
  ObjectType: 0x2::deny_list::DenyList
  Version: <VERSION-NUMBER>
  Digest: <DIGEST-HASH>

...
```
