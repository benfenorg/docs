# 链上随机性

在 Move 中生成伪随机值与其他语言中的解决方案类似。 Move 函数可以创建 `RandomGenerator` 的新实例，并使用它生成不同类型的随机值，例如 `generate_u128(&mut generator)`, `generate_u8_in_range(&mut generator, 1, 6)`：

```move
entry fun roll_dice(r: &Random, ctx: &mut TxContext): Dice {
  let generator = new_generator(r, ctx); // generator is a PRG
  Dice { value: random::generate_u8_in_range(&mut generator, 1, 6) }
}
```

`Random` 有一个保留地址 `0x8` 。请参阅 `random.move` 了解用于访问 Sui 上随机性的 Move API。

:::note
尽管 `Random` 是共享对象，但可变操作无法访问它，并且任何尝试修改它的事务都会失败。
:::

访问随机数只是设计安全应用程序的一部分，您还应该仔细注意如何使用随机性。要安全地访问随机性：

- 将您的函数定义为 (private) `entry` 。
- 推荐使用局部函数 `RandomGenerator` 生成随机性。
- 确保你的函数的“不推荐的路径”不会比“推荐的路径”收取更多的gas。

## 使用非公开 entry 函数​

虽然组合对于智能合约来说非常强大，但它为攻击使用随机性的函数打开了大门。例如考虑下一个模块：

```move
module games::dice {
...
  struct GuessedCorrectly has drop { ... };

  /// If you guess correctly the output you get a GuessedCorrectly object.
  public fun play_dice(guess: u8, fee: Coin<SUI>, r: &Random, ctx: &mut TxContext): Option<GuessedCorrectly> {
    // Pay for the turn
    assert!(coin::value(&fee) == 1000000, EInvalidAmount);
    transfer::public_transfer(fee, CREATOR_ADDRESS);
    // Roll the dice
    let generator = new_generator(r, ctx);
    if (guess == random::generate_u8_in_range(&mut generator, 1, 6)) {
      option::some(GuessedCorrectly {})
    } else {
      option::none()
    }
  }
...
}
```

攻击者可以部署以下功能：

```move
public fun attack(guess: u8, r: &Random, ctx: &mut TxContext): GuessedCorrectly {
  let output = dice::play_dice(guess, r, ctx);
  option::extract(output) // reverts the transaction if roll_dice returns option::none()
}
```

攻击者现在可以通过猜测调用 `attack` ，并且如果猜测不正确，则始终恢复费用转移。

为了防止本示例中的组合攻击，请将 `play_dice` 定义为私有 `entry` 函数，以便其他模块的函数无法调用它，例如：

```move
entry fun play_dice(guess: u8, fee: Coin<SUI>, r: &Random, ctx: &mut TxContext): Option<GuessedCorrectly> {
  ...
}
```

:::note
Move 编译器通过拒绝以 Random 作为参数的 public 函数来强制执行此行为。
:::

## 可编程事务块（PTB）限制​

即使 `play_dice` 被定义为私有 entry 函数，与之前描述的攻击类似的攻击也涉及 PTB。例如，考虑前面定义的 `entry play_dice(guess: u8, fee: Coin<SUI>, r: &Random, ctx: &mut TxContext): Option<GuessedCorrectly> { … }` 函数，攻击者可以发布该函数：

```move
public fun attack(output: Option<GuessedCorrectly>): GuessedCorrectly {
  option::extract(output)
}
```

并发送带有命令 `play_dice(...), attack(Result(0))` 的 PTB，其中 `Result(0)` 是第一个命令的输出。和以前一样，攻击利用了 PTB 的原子性质，如果猜测不正确，总是会恢复整个交易，而无需支付费用。发送多笔交易可以重复攻击，每笔交易都以不同的随机性执行，如果猜测不正确则恢复。

:::note
为了防止基于 PTB 的组合攻击，Sui 会拒绝在使用 Random 或 MergeCoins 命令的 PTB 作为输入。
:::

## 实例化 RandomGenerator ​

`RandomGenerator` 只要是由使用模块创建的，就是安全的。如果作为参数传递，调用者可能能够预测该 `RandomGenerator` 实例的输出（例如，通过调用 `bcs::to_bytes(&generator)` 并解析其内部状态）。

:::note
Move 编译器通过拒绝以 `RandomGenerator` 作为参数的 `public` 函数来强制执行此行为。
:::

### 有限的资源和 Random 依赖流程

开发人员应该意识到，可用于事务的某些资源是有限的。如果读取 `Random` 的函数在不愉快的路径流程中比在愉快的路径流程中消耗更多的资源，则攻击者可以利用该差异来恢复不愉快的流程中的事务，如前所述。具体来说，Gas 就是这样一种资源。考虑以下代码：

```move
// Insecure implementation, do not use.
entry fun insecure_play(r: &Random, payment: Coin<SUI>, ...) {
  ...
  let generator = new_generator(r, ctx);
  let win = random::generate_bool(generator);
  if (win) { // happy flow
    ... cheap computation ...
  } else {
    ... very expensive computation ...
  }
}
```

观察到调用 `insecure_play` 的交易的 Gas 成本取决于 win 的值 - 攻击者可以使用具有足够余额来覆盖快乐流的 Gas 对象来调用此函数，但是不是不高兴的一方，导致它赢得或恢复交易（但绝不会丢失付款）。

在许多情况下，这不是问题，例如在选择抽奖者、彩票号码或随机 NFT 时。但是，在可能出现问题的情况下，您可以执行以下操作之一：

- 以一种正常的流程比不正常的流程消耗更多 Gas 的方式编写函数。
    - 请记住，外部函数或本机函数将来可能会发生变化，与您进行测试的时间相比，可能会导致不同的成本。
    - 在测试网交易上使用 profile-transaction 来验证不同流程的成本。
- 将逻辑分为两部分：一个函数获取随机值并将其存储在对象中，另一个函数读取存储的值并完成操作。后一个函数可能确实会失败，但现在随机值是固定的，无法使用重复调用进行修改。

有关示例，请参阅 `random_nft`。

每笔交易的其他有限资源包括：

- 新对象的数量。
- 可以使用的对象数量（包括动态字段）。

## 从 TypeScript 访问 Random ​

如果您想在模块 `example` 中调用 `roll_dice(r: &Random, ctx: &mut TxContext)` ，请使用以下代码片段：

```typescript
const txb = new Transaction();
txb.moveCall({
  target: "${PACKAGE_ID}::example::roll_dice",
  arguments: [txb.object('0x8')]
});
...
```
