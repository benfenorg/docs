# 忠诚度代币

使用 Bfc Closed-Loop Token 标准，您可以创建仅对特定服务有效的 Token ，例如航空公司想要向常旅客授予 Token 以购买机票或升舱。

以下示例演示了忠诚度代币的创建，持有者可以使用该 Token 在数字礼品店进行购买。

### 示例

忠诚度代币示例说明了使用闭环 Token 标准创建的忠诚度代币 。如果您要实现此示例，管理员将向您的服务用户发送 `LOYALTY` Token ，作为对他们忠诚度的奖励。该示例创建了一个 `GiftShop` ，持有者可以在其中花费 `LOYALTY` 代币购买 `Gift` 。

#### examples::loyalty

`loyalty.move` 源文件包含创建忠诚度令牌的 `examples::loyalty` 模块代码。该模块包括创建硬币的一次性见证人（OTW）（与模块同名 `LOYALTY` ），仅具有 `drop` 能力，并且没有字段。这些是 OTW 的特征，它确保 `LOYALTY` 类型具有单个实例。

```
// examples/move/token/sources/loyalty.move

public struct LOYALTY has drop {}
```

模块的 `init` 函数使用 `LOYALTY` OTW 创建令牌。所有 `init` 函数仅在包发布事件中运行一次。初始化函数使用之前在调用 `create_currency` 时定义的 OTW `LOYALTY` 类型。该功能还定义策略，将策略功能和财务功能发送到与发布事件关联的地址。这些可转让能力的持有者可以铸造新的 `LOYALTY` 代币并修改其政策。

```
// examples/move/token/sources/loyalty.move

fun init(otw: LOYALTY, ctx: &mut TxContext) {
    let (treasury_cap, coin_metadata) = coin::create_currency(
        otw,
        0, // no decimals
        b"LOY", // symbol
        b"Loyalty Token", // name
        b"Token for Loyalty", // description
        option::none(), // url
        ctx
    );

    let (mut policy, policy_cap) = token::new_policy(&treasury_cap, ctx);

    token::add_rule_for_action<LOYALTY, GiftShop>(
        &mut policy,
        &policy_cap,
        token::spend_action(),
        ctx
    );

    token::share_policy(policy);

    transfer::public_freeze_object(coin_metadata);
    transfer::public_transfer(policy_cap, tx_context::sender(ctx));
    transfer::public_transfer(treasury_cap, tx_context::sender(ctx));
}
```

`LOYALTY` 铸造函数称为 `reward_user` 。如前所述， `TreasuryCap` 的持有者可以调用此函数来铸造新的忠诚度代币并将其发送到所需的地址。该函数使用 `token::mint` 函数创建令牌，并使用 `token::transfer` 将其发送给预期接收者。

```
// examples/move/token/sources/loyalty.move

public fun reward_user(
    cap: &mut TreasuryCap<LOYALTY>,
    amount: u64,
    recipient: address,
    ctx: &mut TxContext
) {
    let token = token::mint(cap, amount, ctx);
    let req = token::transfer(token, recipient, ctx);

    token::confirm_with_treasury_cap(cap, req, ctx);
}
```

最后，该示例包含一个 `buy_a_gift` 函数来处理 `Gift` 类型的 `LOYALTY` 令牌的兑换。该函数确保礼品价格与花费的忠诚度代币数量相匹配，然后使用 `token::spend` 函数来处理财务簿记。

```
// examples/move/token/sources/loyalty.move

public fun buy_a_gift(
    token: Token<LOYALTY>,
    ctx: &mut TxContext
): (Gift, ActionRequest<LOYALTY>) {
    assert!(token::value(&token) == GIFT_PRICE, EIncorrectAmount);

    let gift = Gift { id: object::new(ctx) };
    let mut req = token::spend(token, ctx);

    token::add_approval(GiftShop {}, &mut req, ctx);

    (gift, req)
}
```
