# 游戏内代币

使用 Sui Closed-Loop Token 标准，您可以创建游戏内代币（例如手机游戏中的宝石或钻石），您可以将其授予玩家进行操作或可供购买。你在 Sui 上铸造代币，但玩家只能在游戏本身的经济范围内使用代币。这些类型的代币通常不可转让，您通常会按预定数量铸造它们，以保持稀缺性和游戏平衡。

以下示例创建了一种称为 GEM 的游戏内货币，它代表一定数量的 SUI。在示例中，用户可以使用 SUI 购买可替代的 GEM，然后将其用作游戏中的货币。使用代码注释来遵循示例的逻辑。

### 示例

Sui 存储库提供了创建游戏内货币的基本示例。创建示例经济性的 Move 模块位于 gems.move 源文件中。

#### 模块 examples::sword

`examples::sword` 模块创建具有游戏内价值的对象之一 `sword` 。该模块为剑分配了 GEM（游戏中其他有价值的物品）的价值。该模块还提供了交易 GEM 以获得剑的逻辑。

```
// examples/move/token/sources/gems.move

module examples::sword {
    use sui::token::{Self, Token, ActionRequest};
    use examples::gem::GEM;

    const EWrongAmount: u64 = 0;

    const SWORD_PRICE: u64 = 10;

    public struct Sword has key, store { id: UID }

    public fun buy_sword(
        gems: Token<GEM>, ctx: &mut TxContext
    ): (Sword, ActionRequest<GEM>) {
        assert!(SWORD_PRICE == token::value(&gems), EWrongAmount);
        (
            Sword { id: object::new(ctx) },
            token::spend(gems, ctx)
        )
    }
}
```

#### 模块 examples::gem

`examples::gem` 模块创建游戏内货币 GEM。用户花费SUI购买GEM，然后可以将其交易为剑。该模块定义了三组 GEM（小型、中型和大型），每组代表不同的游戏内值。常量保存每个包的值以及组中包含的 GEM 的实际数量。

```
// examples/move/token/sources/gems.move

fun init(otw: GEM, ctx: &mut TxContext) {
    let (treasury_cap, coin_metadata) = coin::create_currency(
        otw, 0, b"GEM", b"Capy Gems", // otw, decimal, symbol, name
        b"In-game currency for Capy Miners", none(), // description, url
        ctx
    );

    let (mut policy, cap) = token::new_policy(&treasury_cap, ctx);

    token::allow(&mut policy, &cap, buy_action(), ctx);
    token::allow(&mut policy, &cap, token::spend_action(), ctx);

    transfer::share_object(GemStore {
        id: object::new(ctx),
        gem_treasury: treasury_cap,
        profits: balance::zero()
    });

    transfer::public_freeze_object(coin_metadata);
    transfer::public_transfer(cap, ctx.sender());
    token::share_policy(policy);
}
```

该模块使用 `buy_gems` 功能处理 GEM 的购买。

```
// examples/move/token/sources/gems.move

public fun buy_gems(
    self: &mut GemStore, payment: Coin<SUI>, ctx: &mut TxContext
): (Token<GEM>, ActionRequest<GEM>) {
    let amount = coin::value(&payment);
    let purchased = if (amount == SMALL_BUNDLE) {
        SMALL_AMOUNT
    } else if (amount == MEDIUM_BUNDLE) {
        MEDIUM_AMOUNT
    } else if (amount == LARGE_BUNDLE) {
        LARGE_AMOUNT
    } else {
        abort EUnknownAmount
    };

    coin::put(&mut self.profits, payment);

    let gems = token::mint(&mut self.gem_treasury, purchased, ctx);
    let req = token::new_request(buy_action(), purchased, none(), none(), ctx);

    (gems, req)
}
```
