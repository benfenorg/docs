# 编译并测试 Move 包

如果您已经完成了“编写 Move 包”，则您已经有一个需要构建的基本模块。

### 构建你的包

确保您的终端或控制台位于包含您的包的目录中（ `my_first_package` ，如果您正在执行此操作）。使用以下命令来构建您的包：

```
$ bfc move build
```

成功的构建会返回类似于以下内容的响应：

```
UPDATING GIT DEPENDENCY https://github.com/MystenLabs/bfc.git
INCLUDING DEPENDENCY Bfc
INCLUDING DEPENDENCY MoveStdlib
BUILDING my_first_package
```

如果构建失败，您可以使用输出中的详细错误消息来排除故障并解决根本问题。

现在您已经设计了资产及其访问器函数，是时候在发布之前测试包代码了。

###  测试包​

Bfc 包括对 Move 测试框架的支持。使用该框架，您可以编写分析 Move 代码的单元测试，就像其他语言的测试框架一样，例如内置的 Rust 测试框架或 Java 的 JUnit 框架。

单个 Move 单元测试封装在一个公共函数中，该函数没有参数、没有返回值，并且具有 `#[test]` 注释。当您从包根目录（根据当前运行示例的 `my_move_package` 目录）调用 `bfc move test` 命令时，测试框架会执行此类函数：

```
$ bfc move test
```

如果对编写包中创建的包执行此命令，您将看到以下输出。毫不奇怪，测试结果具有 `OK` 状态，因为还没有编写失败的测试。

```
BUILDING Bfc
BUILDING MoveStdlib
BUILDING my_first_package
Running Move unit tests
Test result: OK. Total tests: 0; passed: 0; failed: 0
```

要实际测试您的代码，您需要添加测试函数。首先将基本测试函数添加到模块定义内的 `my_module.move` 文件中：

```
// examples/move/first_package/sources/example.move
#[test]
fun test_sword_create() {
    // Create a dummy TxContext for testing
    let mut ctx = tx_context::dummy();

    // Create a sword
    let sword = Sword {
        id: object::new(&mut ctx),
        magic: 42,
        strength: 7,
    };

    // Check if accessor functions return correct values
    assert!(sword.magic() == 42 && sword.strength() == 7, 1);
}
```

如代码所示，单元测试函数 ( `test_sword_create()` ) 创建 `TxContext` 结构的虚拟实例并将其分配给 `ctx` 。然后，该函数使用 `ctx` 创建一个剑对象来创建唯一标识符，并将 `42` 分配给 `magic` 参数，将 `7` 分配给 `strength` 。最后，测试调用 `magic` 和 `strength` 访问器函数来验证它们是否返回正确的值。

该函数将虚拟上下文 `ctx` 作为可变引用参数 ( `&mut` ) 传递给 `object::new` 函数，但将 `sword` 传递给它的访问器用作只读引用参数 `&sword` 。

现在您已经有了测试功能，请再次运行测试命令：

```
$ bfc move test
```

但是，运行 `test` 命令后，您会收到编译错误而不是测试结果：

```
error[E06001]: unused value without 'drop'
   ┌─ ./sources/my_module.move:59:65
   │
 9 │       public struct Sword has key, store {
   │                     ----- To satisfy the constraint, the 'drop' ability would need to be added here
   ·
52 │           let sword = Sword {
   │               ----- The local variable 'sword' still contains a value. The value does not have the 'drop' ability and must be consumed before the function returns
   │ ╭─────────────────────'
53 │ │             id: object::new(&mut ctx),
54 │ │             magic: 42,
55 │ │             strength: 7,
56 │ │         };
   │ ╰─────────' The type 'my_first_package::my_module::Sword' does not have the ability 'drop'
   · │
59 │           assert!(sword.magic() == 42 && sword.strength() == 7, 1);
   │                                                                   ^ Invalid return
```

错误消息包含调试代码所需的所有信息。错误代码旨在突出 Move 语言的一项安全功能。

`Sword` 结构代表以数字方式模仿现实世界物品的游戏资产。显然，一把真正的剑不能简单地消失（尽管它可以被明确地摧毁），但数字剑则没有这样的限制。事实上，这正是 `test` 函数中发生的情况 - 您创建了 `Sword` 结构的实例，该实例在函数调用结束时消失。如果你看到有东西在你眼前消失，你也会目瞪口呆。

解决方案之一（如错误消息中建议的那样）是将 `drop` 功能添加到 `Sword` 结构的定义中，这将允许该结构的实例消失（可以是掉落）。在这种情况下，丢弃有价值的资产的能力并不是理想的资产属性，因此需要另一种解决方案。解决此问题的另一种方法是转移 `sword` 的所有权。

为了使测试正常工作，我们需要使用默认导入的 `transfer` 模块。将以下行添加到测试函数的末尾（在 `assert!` 调用之后），以将 `sword` 的所有权转移到新创建的虚拟地址：

```
// examples/move/first_package/sources/example.move
let dummy_address = @0xCAFE;
transfer::public_transfer(sword, dummy_address);
```

再次运行测试命令。现在输出显示单个成功的测试已运行：

```
BUILDING MoveStdlib
BUILDING Bfc
BUILDING my_first_package
Running Move unit tests
[ PASS    ] 0x0::my_module::test_sword_create
Test result: OK. Total tests: 1; passed: 1; failed: 0
```

:::tip
使用过滤字符串仅运行单元测试的匹配子集。通过提供的过滤字符串， `bfc move test` 检查完全限定 ( `<address>::<module_name>::<fn_name>` ) 名称是否匹配。
:::

例子：

```
$ bfc move test sword
```

上一个命令运行名称包含 `sword` 的所有测试。

您可以通过以下方式发现更多测试选项：

```
$ bfc move test -h
```

### Bfc 特定测试​

前面的测试示例使用 Move，但除了使用一些 Bfc 包（例如 `bfc::tx_context` 和 `bfc::transfer` ）之外，并不特定于 Sui。虽然这种测试风格对于为 Bfc 编写 Move 代码已经很有用，但您可能还想测试其他特定于 Bfc 的功能。特别是，Sui 中的 Move 调用封装在 Bfc 事务中，您可能希望在单个测试中测试不同事务之间的交互（例如，一个事务创建对象，另一个事务传输对象）。

Bfc 特定的测试是通过 `test_scenario` 模块支持的，该模块提供了与 Bfc 相关的测试功能，而这些功能在纯 Move 及其测试框架中是不可用的。

`test_scenario` 模块提供了一个模拟一系列 Bfc 事务的场景，每个事务都有可能由不同的用户执行。使用此模块的测试通常使用 `test_scenario::begin` 函数启动第一个事务。该函数将执行交易的用户地址作为参数，并返回表示场景的 `Scenario` 结构体的实例。

`Scenario` 结构的实例包含模拟 Bfc 对象存储的每地址对象池，并提供用于操作池中对象的辅助函数。第一个事务完成后，后续的测试事务从 `test_scenario::next_tx` 函数开始。该函数采用表示当前场景的 `Scenario` 结构实例和用户地址作为参数。

更新您的 `my_module.move` 文件以包含可从 Bfc 调用的实现 `sword` 创建的函数。完成此操作后，您就可以添加使用 `test_scenario` 模块的多事务测试来测试这些新功能。将此函数放在访问器之后（注释中的第 5 部分）。

```
// examples/move/first_package/sources/example.move

public fun sword_create(magic: u64, strength: u64, ctx: &mut TxContext): Sword {
    Sword {
        id: object::new(ctx),
        magic: magic,
        strength: strength,
    }
}
```

新函数的代码使用结构体创建和 Bfc 内部模块 ( `tx_context` )，其方式与您在前面几节中看到的类似。重要的部分是函数具有正确的签名。

包含新函数后，添加另一个测试函数以确保其行为符合预期。

```
// examples/move/first_package/sources/example.move

#[test]
fun test_sword_transactions() {
    use bfc::test_scenario;

    // Create test addresses representing users
    let initial_owner = @0xCAFE;
    let final_owner = @0xFACE;

    // First transaction executed by initial owner to create the sword
    let mut scenario = test_scenario::begin(initial_owner);
    {
        // Create the sword and transfer it to the initial owner
        let sword = sword_create(42, 7, scenario.ctx());
        transfer::public_transfer(sword, initial_owner);
    };

    // Second transaction executed by the initial sword owner
    scenario.next_tx(initial_owner);
    {
        // Extract the sword owned by the initial owner
        let sword = scenario.take_from_sender<Sword>();
        // Transfer the sword to the final owner
        transfer::public_transfer(sword, final_owner);
    };

    // Third transaction executed by the final sword owner
    scenario.next_tx(final_owner);
    {
        // Extract the sword owned by the final owner
        let sword = scenario.take_from_sender<Sword>();
        // Verify that the sword has expected properties
        assert!(sword.magic() == 42 && sword.strength() == 7, 1);
        // Return the sword to the object pool (it cannot be simply "dropped")
        scenario.return_to_sender(sword)
    };
    scenario.end();
}
```

新的测试功能有一些细节需要注意。代码所做的第一件事是创建一些代表参与测试场景的用户的地址。然后，测试通过代表初始剑拥有者启动第一笔交易来创建一个场景。

然后，初始所有者执行第二笔交易（作为参数传递给 `test_scenario::next_tx` 函数），后者将他们现在拥有的 `sword` 转移给最终所有者。在纯粹的Move中，没有Sui存储的概念；因此，模拟的 Bfc 交易没有简单的方法从存储中检索它。这就是 `test_scenario` 模块提供帮助的地方 - 它的 `take_from_sender` 函数允许执行当前事务的给定类型（ `Sword` ）的地址拥有的对象可用于移动代码操作。现在，假设只有一个这样的对象。在这种情况下，测试将从存储中检索的对象传输到另一个地址。

:::tip
事务效果（例如对象创建和传输）仅在给定事务完成后才可见。例如，如果运行示例中的第二个事务创建了 `sword` 并将其传输到管理员的地址，则它只能从管理员的地址中检索（通过 `test_scenario` 、`< b2>` 或 `take_from_address` 函数）在第三个事务中。
:::

最终所有者执行第三个也是最后一个事务，从存储中检索 `sword` 对象并检查它是否具有预期的属性。请记住，如测试包中所述，在纯 Move 测试场景中，当对象在 Move 代码中可用后（在创建或从模拟存储中检索后），它不能简单地消失。

在纯Move测试函数中，该函数将 `sword` 对象传输到假地址来处理消失问题。然而， `test_scenario` 包提供了一个更优雅的解决方案，它更接近 Move 代码在 Bfc 上下文中实际执行时发生的情况 - 该包只是将 `sword` 返回到对象池使用 `test_scenario::return_to_sender` 函数。对于不希望返回发送者或者您只想销毁对象的情况， `test_utils` 模块还提供通用 `destroy<T>` 函数，该函数可用于任何类型 T 无论其能力如何。建议还检查 `test_scenario` 和 `test_utils` 模块中的其他有用功能。

再次运行测试命令可以看到我们的模块的两次成功测试：

```
BUILDING Bfc
BUILDING MoveStdlib
BUILDING my_first_package
Running Move unit tests
[ PASS    ] 0x0::my_module::test_sword_create
[ PASS    ] 0x0::my_module::test_sword_transactions
Test result: OK. Total tests: 2; passed: 2; failed: 0
```

###  模块初始化

包中的每个模块都可以包含一个在发布时运行的特殊初始化函数。初始化函数的目标是预初始化特定于模块的数据（例如，创建单例对象）。初始化函数必须具有以下属性才能在发布时执行：

* 函数名称必须是 `init`
* 参数列表必须以 `&mut TxContext` 或 `&TxContext` 类型结尾
* 无返回值
* 私有可见
* （可选）参数列表首先接受模块的一次性见证值

例如，以下 `init` 函数都是有效的：

- `fun init(ctx: &TxContext)`
- `fun init(ctx: &mut TxContext)`
- `fun init(otw: EXAMPLE, ctx: &TxContext)`
- `fun init(otw: EXAMPLE, ctx: &mut TxContext)`

虽然 `bfc move` 命令不支持显式发布，但您仍然可以使用测试框架测试模块初始值设定项，方法是将第一个事务专用于执行初始值设定项函数。

运行示例中模块的 `init` 函数创建一个 `Forge` 对象。

```
// examples/move/first_package/sources/example.move

fun init(ctx: &mut TxContext) {
    let admin = Forge {
        id: object::new(ctx),
        swords_created: 0,
    };

    transfer::transfer(admin, ctx.sender());
}
```

到目前为止，您进行的测试调用了 `init` 函数，但初始化函数本身并未经过测试以确保它正确创建 `Forge` 对象。要测试此功能，请添加一个 `new_sword` 函数以将锻造作为参数，并在函数末尾更新创建的剑的数量。如果这是一个实际的模块，您可以将 `sword_create` 函数替换为 `new_sword` 。然而，为了防止现有测试失败，我们将保留这两个功能。

```
// examples/move/first_package/sources/example.move

public fun new_sword(
    forge: &mut Forge,
    magic: u64,
    strength: u64,
    ctx: &mut TxContext,
): Sword {
    forge.swords_created = forge.swords_created + 1;
    Sword {
        id: object::new(ctx),
        magic: magic,
        strength: strength,
    }
}
```

现在，创建一个函数来测试模块初始化：

```
// examples/move/first_package/sources/example.move

#[test]
fun test_module_init() {
    use bfc::test_scenario;

    // Create test addresses representing users
    let admin = @0xAD;
    let initial_owner = @0xCAFE;

    // First transaction to emulate module initialization
    let mut scenario = test_scenario::begin(admin);
    {
        init(scenario.ctx());
    };

    // Second transaction to check if the forge has been created
    // and has initial value of zero swords created
    scenario.next_tx(admin);
    {
        // Extract the Forge object
        let forge = scenario.take_from_sender<Forge>();
        // Verify number of created swords
        assert!(forge.swords_created() == 0, 1);
        // Return the Forge object to the object pool
        scenario.return_to_sender(forge);
    };

    // Third transaction executed by admin to create the sword
    scenario.next_tx(admin);
    {
        let mut forge = scenario.take_from_sender<Forge>();
        // Create the sword and transfer it to the initial owner
        let sword = forge.new_sword(42, 7, scenario.ctx());
        transfer::public_transfer(sword, initial_owner);
        scenario.return_to_sender(forge);
    };
    scenario.end();
}
```

正如新的测试函数所示，第一个事务（显式）调用初始化程序。下一个事务检查 `Forge` 对象是否已创建并正确初始化。最后，管理员使用 `Forge` 创建一把剑并将其转让给初始所有者。

您可以参考 `bfc/examples` 目录下的 `first_package` 模块中的包的源代码（所有测试和功能都经过适当调整）。
