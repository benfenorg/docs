# 访问链上时间

当需要访问基于网络的时间进行交易时，您可以选择。如果您需要近乎实时的测量（几秒内），请使用 Move 中的 Clock 模块提供的不可变时间参考。该模块的参考值随每个网络检查点而更新。如果您不需要当前时间片，请使用 epoch_timestamp_ms 函数捕获当前纪元开始的精确时刻。

### bfc::clock::Clock 模块

要访问提示时间戳，您必须传递 `bfc::clock::Clock` 的只读引用作为事务中的入口函数参数。地址 `0x6` 处提供了 `Clock` 的实例，无法创建新实例。

使用 `bfc::clock` 模块中的 `timestamp_ms` 函数提取以毫秒为单位的 unix 时间戳。

```
// crates/bfc-framework/packages/bfc-framework/sources/clock.move

public fun timestamp_ms(clock: &Clock): u64 {
    clock.timestamp_ms
}
```

下面的示例演示了一个入口函数，它发出一个包含来自 `Clock` 的时间戳的事件：

```
// examples/move/basics/sources/clock.move

module basics::clock {
    use bfc::{clock::Clock, event};

    public struct TimeEvent has copy, drop, store {
        timestamp_ms: u64,
    }

    entry fun access(clock: &Clock) {
        event::emit(TimeEvent { timestamp_ms: clock.timestamp_ms() });
    }
}
```

对前一个入口函数的调用采用以下形式，传递 `0x6` 作为 `Clock` 参数的地址：

```
bfc client call --package <EXAMPLE> --module 'clock' --function 'access' --args '0x6'
```

预计 `Clock` 时间戳会按照网络生成检查点的速率变化，对于 Narwhal/Bullshark 共识，每 1 秒变化一次；对于 Mysticeti 共识，每 0.1 到 0.2 秒变化一次。

在同一事务中连续调用 `bfc::clock::timestamp_ms` 始终会产生相同的结果（事务被视为立即生效），但来自 `Clock` 的时间戳在接触相同共享对象的事务之间是单调的：连续事务的时间戳大于或等于其前一个事务的时间戳。

任何需要访问 `Clock` 的事务都必须经过共识，因为唯一可用的实例是共享对象。因此，此技术不适合必须使用单所有者快速路径的事务（有关单所有者兼容的时间戳源，请参阅 Epoch 时间戳）。

使用时钟的事务必须接受它作为不可变引用（而不是可变引用或值）。这可以防止争用，因为访问 `Clock` 的事务只能读取它，因此不需要相对于彼此进行排序。验证者拒绝签署不满足此要求的交易，并且包含接受 `Clock` 或 `&mut Clock` 的条目函数的包无法发布。

以下函数通过手动创建 `Clock` 对象并操作其时间戳来测试 `Clock` 相关代码。这仅在测试代码中可行：

```
// crates/bfc-framework/packages/bfc-framework/sources/clock.move

#[test_only]
public fun create_for_testing(ctx: &mut TxContext): Clock {
    Clock {
        id: object::new(ctx),
        timestamp_ms: 0,
    }
}

#[test_only]
public fun share_for_testing(clock: Clock) {
    transfer::share_object(clock)
}

#[test_only]
public fun increment_for_testing(clock: &mut Clock, tick: u64) {
    clock.timestamp_ms = clock.timestamp_ms + tick;
}

#[test_only]
public fun set_for_testing(clock: &mut Clock, timestamp_ms: u64) {
    assert!(timestamp_ms >= clock.timestamp_ms);
    clock.timestamp_ms = timestamp_ms;
}

#[test_only]
public fun destroy_for_testing(clock: Clock) {
    let Clock { id, timestamp_ms: _ }  = clock;
    id.delete();
}
```

下一个示例介绍了一个基本测试，该测试创建一个 `Clock`，递增它，然后检查它的值：

```
// crates/bfc-framework/packages/bfc-framework/tests/clock_tests.move

#[test_only]
module bfc::clock_tests {
    use bfc::clock;

    #[test]
    fun creating_a_clock_and_incrementing_it() {
        let mut ctx = tx_context::dummy();
        let mut clock = clock::create_for_testing(&mut ctx);

        clock.increment_for_testing(42);
        assert!(clock.timestamp_ms() == 42);

        clock.set_for_testing(50);
        assert!(clock.timestamp_ms() == 50);

        clock.destroy_for_testing();
    }
}
```

### Epoch 时间戳

使用 `bfc::tx_context` 模块中的以下函数来访问所有交易（包括未经过共识的交易）当前纪元开始的时间戳：

```
// crates/bfc-framework/packages/bfc-framework/sources/tx_context.move

public fun epoch_timestamp_ms(self: &TxContext): u64 {
   self.epoch_timestamp_ms
}
```

前面的函数返回当前纪元开始的时间点，作为 `u64` 中的毫秒粒度 unix 时间戳。当纪元发生变化时，该值大约每 24 小时变化一次。

基于 `bfc::test_scenario` 的测试可以使用 `later_epoch` （以下代码）来练习使用 `epoch_timestamp_ms` （之前的代码）的时间敏感代码：

```
// crates/bfc-framework/packages/bfc-framework/sources/test/test_scenario.move

public fun later_epoch(
    scenario: &mut Scenario,
    delta_ms: u64,
    sender: address,
): TransactionEffects {
    scenario.ctx.increment_epoch_timestamp(delta_ms);
    next_epoch(scenario, sender)
}
```

`later_epoch` 的行为类似于 `bfc::test_scenario::next_epoch` （完成测试场景中的当前事务和纪元），但还将时间戳增加 `delta_ms` 毫秒以模拟时间的进度。
