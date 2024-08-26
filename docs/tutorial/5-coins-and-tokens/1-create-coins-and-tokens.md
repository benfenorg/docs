# 创建 Coin 和 Token

在 BenFen 上发行 Coin 与发行新类型类似。主要区别在于创造 Coin 时需要一次性见证人。

```
// examples/move/coin/sources/my_coin.move

module examples::my_coin {
    use sui::coin::{Self, TreasuryCap};

    public struct MY_COIN has drop {}

    #[allow(lint(share_owned))]
    fun init(witness: MY_COIN, ctx: &mut TxContext) {
        let (treasury, metadata) = coin::create_currency(witness, 6, b"MY_COIN", b"", b"", option::none(), ctx);
        transfer::public_share_object(metadata);
        transfer::public_transfer(treasury, ctx.sender())
    }

    public fun mint(
        treasury_cap: &mut TreasuryCap<MY_COIN>, 
        amount: u64, 
        recipient: address, 
        ctx: &mut TxContext,
    ) {
        let coin = coin::mint(treasury_cap, amount, ctx);
        transfer::public_transfer(coin, recipient)
    }
}
```

`Coin<T>` 是 Bfc 上硬币的通用实现。访问 `TreasuryCap` 可以控制硬币的铸造和燃烧。进一步的交易可以直接发送到 `bfc::coin::Coin` ，并使用 `TreasuryCap` 对象作为授权。

该示例模块包含一个 `mint` 函数。您将从 `init` 函数创建的 `TreasuryCap` 传递给模块的 `mint` 函数。然后，该函数使用 `Coin` 模块中的 `mint` 函数创建（铸造）一枚硬币，然后将其转移到一个地址。

```
// examples/move/coin/sources/my_coin.move

public fun mint(
    treasury_cap: &mut TreasuryCap<MY_COIN>, 
    amount: u64, 
    recipient: address, 
    ctx: &mut TxContext,
) {
    let coin = coin::mint(treasury_cap, amount, ctx);
    transfer::public_transfer(coin, recipient)
}
```

### BenFen CLI

如果您将前面的示例发布到 Bfc 网络，则可以使用 `bfc client call` 命令铸造硬币并将其传送到您提供的地址。有关命令行界面的更多信息，请参阅 Bfc CLI。

```
bfc client call --function mint --module mycoin --package <PACKAGE-ID> --args <TREASURY-CAP-ID> <COIN-AMOUNT> <RECIPIENT-ADDRESS>
```

如果调用成功，您的控制台将显示结果，其中包括余额更改部分，其中包含以下信息：

```
...

Owner: Account Address ( <RECIPIENT-ADDRESS> ) 
CoinType: <PACKAGE-ID>::mycoin::MYCOIN 
Amount: <COIN-AMOUNT>

...
```

###  拒绝名单​

Bfc 框架提供了一个 `DenyList` 单例共享对象， `DenyCap` 的持有者可以访问该对象来指定无法使用 Bfc 核心类型的地址列表。然而， `DenyList` 的初始用例侧重于限制对指定类型硬币的访问。例如，当在 Bfc 上创建受监管的代币时，需要能够阻止某些地址将其用作交易的输入，这非常有用。 Bfc 上受监管的代币满足任何要求能够防止已知不良行为者获取这些代币的法规。

:::info
DenyList 对象是一个系统对象，其地址为 `0x403` 。您无法自己创建它。
:::

### 创建受监管的 Coin

如果您需要能够拒绝特定地址访问您的代币，您可以使用 `create_regulated_currency` 函数（而不是 `create_currency` ）来创建它。

在幕后， `create_regulated_currency` 使用 `create_currency` 函数创建代币，同时还生成一个 `DenyCap` 对象，允许其持有者控制对代币拒绝列表的访问在 `DenyList` 对象中。因此，使用 `create_regulated_currency` 创建硬币的方式与前面的示例类似，只是将 `DenyCap` 对象传输到模块发布者。

类似 Coin 的功能执行代币的铸造和燃烧。两者都需要 TreasuryCap ：

- `token::mint` - 铸造一个代币
- `token::burn` - 烧毁代币
