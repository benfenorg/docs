# 第一个Move程序

在本章中，您将学习如何创建新包、编写简单模块、编译它以及使用 Move CLI 运行测试。确保你已经安装了BenFen并设置您的IDE 环境。运行以下命令测试BenFen是否已正确安装。

```plain
bfc client --version
```

## 创建新工程

创建一个新的工程，实际上就是创建一个Move包。使用`bfc move new`命令，后接包名，此处为`hello_world`：

```plain
$ bfc move new hello_world
```

我们可以查看该文件夹的内容，具有以下目录结构：

```plain
$ tree hello_world 
hello_world
├── Move.toml
├── sources
│   └── hello_world.move
└── tests
    └── hello_world_tests.move

2 directories, 3 files
```

其中，`Move.toml`是Move工程的配置文件，`sources`目录包含Move源代码，`tests`目录包含Move测试代码。

## 编写Move代码

我们打开`hello_world.move`文件，在其中编写如下的代码：

```rust
// 模块声明:
module hello_world::hello_world;

// 导入标准库的模块:
use std::string;

// 入口函数:
public entry fun hello(): string::String {
    string::utf8(b"Hello, World!")
}
```

上述代码首先声明了一个模块`hello_world::hello_world`，紧接着导入了后续要使用的`std::string`模块，然后定义了一个名为`hello`的入口函数，该函数返回一个`Hello, World!`字符串。

运行`bfc move build`命令来编译Move代码：

```plain
$ bfc move build
UPDATING GIT DEPENDENCY https://github.com/MystenLabs/sui.git
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING hello_world
```

如果没有错误输出，则表示编译成功。下一步就可以把它发布到链上。

不过，在发布之前，我们最好在本地对Move代码进行测试。Move提供了一套独立的本地测试环境，可以方便地对函数进行测试。

## 编写测试代码

测试代码必须存放在`tests`目录下。我们打开`hello_world_tests.move`文件，在其中编写如下的测试代码：

```rust
#[test_only]
module hello_world::hello_world_tests;

use hello_world::hello_world;

#[test]
fun test_hello_world() {
    assert!(hello_world::hello() == b"Hello, World!".to_string(), 0);
}
```

首行`#[test_only]`表示该模块仅用于测试，而不是用于生产环境。

接下来仍然是声明模块，不过此处声明的模块并不是`hello_world::hello_world`，而是`hello_world::hello_world_tests`。

然后，我们要导入之前编写的`hello_world::hello_world`模块。

接下来，我们编写测试函数。一个测试函数总是用`#[test]`注解，并且函数名以`test_`开头。函数不接受任何参数，也不返回任何值。

在函数内部，调用待测试的函数，并使用`assert!`宏来检查函数的返回值是否符合预期。

运行`bfc move test`命令来运行测试代码：

```plain
$ bfc move test
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING hello_world
Running Move unit tests
[ PASS    ] hello_world::hello_world_tests::test_hello_world
Test result: OK. Total tests: 1; passed: 1; failed: 0
```

测试输出OK表示测试成功。

如果修改了源代码的返回值，则测试会失败：

```plain
$ bfc move test
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING hello_world
Running Move unit tests
[ FAIL    ] hello_world::hello_world_tests::test_hello_world

Test failures:

Failures in hello_world::hello_world_tests:

┌── test_hello_world ──────
│ error[E11001]: test failure
│   ┌─ ./tests/hello_world_tests.move:8:5
│   │
│ 7 │ fun test_hello_world() {
│   │     ---------------- In this function in hello_world::hello_world_tests
│ 8 │     assert!(hello_world::hello() == b"Hello, World!".to_string(), 0);
│   │     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Test was not expected to error, but it aborted ...
│ 
│ 
└──────────────────

Test result: FAILED. Total tests: 1; passed: 0; failed: 1
```

可见，本地测试能帮助我们快速发现代码问题，而不必等待部署至链上后再测试。

## 发布Move模块

修复代码后，我们可以使用`bfc client publish`命令将模块发布到链上：

```plain
$ bfc client publish
UPDATING GIT DEPENDENCY https://github.com/MystenLabs/sui.git
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING hello_world
Successfully verified dependencies on-chain against source.
Transaction Digest: DQBGth6dS6F9dP3ntbLJhN7sxJdVWoHrjDzSSNsH1vYN
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Transaction Data                                                                                             │
├──────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Sender: 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767                                   │
│ Gas Owner: 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767                                │
│ Gas Budget: 7905200 MIST                                                                                     │
│ Gas Price: 1000 MIST                                                                                         │
│ Gas Payment:                                                                                                 │
│  ┌──                                                                                                         │
│  │ ID: 0x5085ad52a308634fbb2dc7cd25177475c8bc99b2c48804cc3e838d2462b67f7d                                    │
│  │ Version: 92487848                                                                                         │
│  │ Digest: 2AfKuvprhm5iHyyW1YQ6q3Wt3KiKB8Pjipe1ACfbMiD5                                                      │
│  └──                                                                                                         │
│                                                                                                              │
│ Transaction Kind: Programmable                                                                               │
│ ╭──────────────────────────────────────────────────────────────────────────────────────────────────────────╮ │
│ │ Input Objects                                                                                            │ │
│ ├──────────────────────────────────────────────────────────────────────────────────────────────────────────┤ │
│ │ 0   Pure Arg: Type: address, Value: "0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767" │ │
│ ╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯ │
│ ╭─────────────────────────────────────────────────────────────────────────╮                                  │
│ │ Commands                                                                │                                  │
│ ├─────────────────────────────────────────────────────────────────────────┤                                  │
│ │ 0  Publish:                                                             │                                  │
│ │  ┌                                                                      │                                  │
│ │  │ Dependencies:                                                        │                                  │
│ │  │   0x0000000000000000000000000000000000000000000000000000000000000001 │                                  │
│ │  │   0x0000000000000000000000000000000000000000000000000000000000000002 │                                  │
│ │  └                                                                      │                                  │
│ │                                                                         │                                  │
│ │ 1  TransferObjects:                                                     │                                  │
│ │  ┌                                                                      │                                  │
│ │  │ Arguments:                                                           │                                  │
│ │  │   Result 0                                                           │                                  │
│ │  │ Address: Input  0                                                    │                                  │
│ │  └                                                                      │                                  │
│ ╰─────────────────────────────────────────────────────────────────────────╯                                  │
│                                                                                                              │
│ Signatures:                                                                                                  │
│    y1BfGUdjnZV2VBwpkAKJqtbJPF/YB8+H4w+BbeAasHt08fc8LQeikn8ETwJ6M5rdcR0xaptx4iZgzjmk7B5GBg==                  │
│                                                                                                              │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Transaction Effects                                                                               │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Digest: DQBGth6dS6F9dP3ntbLJhN7sxJdVWoHrjDzSSNsH1vYN                                              │
│ Status: Success                                                                                   │
│ Executed Epoch: 571                                                                               │
│                                                                                                   │
│ Created Objects:                                                                                  │
│  ┌──                                                                                              │
│  │ ID: 0x323bd5ddafa8e15c9aec735b2c34b28198caee69d6657fd0cb8021b88a150967                         │
│  │ Owner: Immutable                                                                               │
│  │ Version: 1                                                                                     │
│  │ Digest: 2aMWMYRyCE2hdbffYCjm6NT4UHE3Nedz4SKzKJKKkqrN                                           │
│  └──                                                                                              │
│  ┌──                                                                                              │
│  │ ID: 0x94770db01865117ab24a5a09845a9571d89797e248a48e11ba60f6cfc8be6932                         │
│  │ Owner: Account Address ( 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767 )  │
│  │ Version: 92487849                                                                              │
│  │ Digest: HCSMNVDXYDUpmq526sTZuFFm9ePJKhFi6ZhPukR7TWMS                                           │
│  └──                                                                                              │
│ Mutated Objects:                                                                                  │
│  ┌──                                                                                              │
│  │ ID: 0x5085ad52a308634fbb2dc7cd25177475c8bc99b2c48804cc3e838d2462b67f7d                         │
│  │ Owner: Account Address ( 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767 )  │
│  │ Version: 92487849                                                                              │
│  │ Digest: DXkDbTxjc9GrCbHsMrsjy5B1vmsd4qVCov49tnxCYQTK                                           │
│  └──                                                                                              │
│ Gas Object:                                                                                       │
│  ┌──                                                                                              │
│  │ ID: 0x5085ad52a308634fbb2dc7cd25177475c8bc99b2c48804cc3e838d2462b67f7d                         │
│  │ Owner: Account Address ( 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767 )  │
│  │ Version: 92487849                                                                              │
│  │ Digest: DXkDbTxjc9GrCbHsMrsjy5B1vmsd4qVCov49tnxCYQTK                                           │
│  └──                                                                                              │
│ Gas Cost Summary:                                                                                 │
│    Storage Cost: 5905200 MIST                                                                     │
│    Computation Cost: 1000000 MIST                                                                 │
│    Storage Rebate: 978120 MIST                                                                    │
│    Non-refundable Storage Fee: 9880 MIST                                                          │
│                                                                                                   │
│ Transaction Dependencies:                                                                         │
│    79wSNGTEKPDffiyBQDm11NiAxbj8fJJuHDioDeC4VatL                                                   │
│    GMBJA2gEEvtwv1wGGT7ZEDkQdrmUTKaE4TeinNGQ2feC                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─────────────────────────────╮
│ No transaction block events │
╰─────────────────────────────╯

╭──────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Object Changes                                                                                   │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Created Objects:                                                                                 │
│  ┌──                                                                                             │
│  │ ObjectID: 0x94770db01865117ab24a5a09845a9571d89797e248a48e11ba60f6cfc8be6932                  │
│  │ Sender: 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767                    │
│  │ Owner: Account Address ( 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767 ) │
│  │ ObjectType: 0x2::package::UpgradeCap                                                          │
│  │ Version: 92487849                                                                             │
│  │ Digest: HCSMNVDXYDUpmq526sTZuFFm9ePJKhFi6ZhPukR7TWMS                                          │
│  └──                                                                                             │
│ Mutated Objects:                                                                                 │
│  ┌──                                                                                             │
│  │ ObjectID: 0x5085ad52a308634fbb2dc7cd25177475c8bc99b2c48804cc3e838d2462b67f7d                  │
│  │ Sender: 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767                    │
│  │ Owner: Account Address ( 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767 ) │
│  │ ObjectType: 0x2::coin::Coin<0x2::sui::SUI>                                                    │
│  │ Version: 92487849                                                                             │
│  │ Digest: DXkDbTxjc9GrCbHsMrsjy5B1vmsd4qVCov49tnxCYQTK                                          │
│  └──                                                                                             │
│ Published Objects:                                                                               │
│  ┌──                                                                                             │
│  │ PackageID: 0x323bd5ddafa8e15c9aec735b2c34b28198caee69d6657fd0cb8021b88a150967                 │
│  │ Version: 1                                                                                    │
│  │ Digest: 2aMWMYRyCE2hdbffYCjm6NT4UHE3Nedz4SKzKJKKkqrN                                          │
│  │ Modules: hello_world                                                                          │
│  └──                                                                                             │
╰──────────────────────────────────────────────────────────────────────────────────────────────────╯
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Balance Changes                                                                                   │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│  ┌──                                                                                              │
│  │ Owner: Account Address ( 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767 )  │
│  │ CoinType: 0x2::sui::SUI                                                                        │
│  │ Amount: -5927080                                                                               │
│  └──                                                                                              │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
```

发布至测试链后，我们观察输出，找到Published Objects：

```plain
PackageID: 0x323bd5ddafa8e15c9aec735b2c34b28198caee69d6657fd0cb8021b88a150967
Version: 1
Digest: 2aMWMYRyCE2hdbffYCjm6NT4UHE3Nedz4SKzKJKKkqrN
Modules: hello_world
PackageID是包的唯一地址。这样我们就得到了一个已发布的模块，它的全名为0x323b...0967::hello_word。
```

### 调用函数

要调用已发布至链上的模块，我们需要指定函数的全路径`PackageID::module::function`。其中，`PackageID`是发布时返回的包地址，此处的module名为`hello_world`，函数名为`hello`。

调用命令如下：

```plain
$ bfc client ptb --move-call 0x323bd5ddafa8e15c9aec735b2c34b28198caee69d6657fd0cb8021b88a150967::hello_world::hello
Transaction Digest: EMqwhupyxNM7MBEfhLGDvTLmE8YHdzASEzoBrXhWKNJb
╭─────────────────────────────────────────────────────────────────────────────────────────────╮
│ Transaction Data                                                                            │
├─────────────────────────────────────────────────────────────────────────────────────────────┤
│ Sender: 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767                  │
│ Gas Owner: 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767               │
│ Gas Budget: 2988000 MIST                                                                    │
│ Gas Price: 1000 MIST                                                                        │
│ Gas Payment:                                                                                │
│  ┌──                                                                                        │
│  │ ID: 0x5085ad52a308634fbb2dc7cd25177475c8bc99b2c48804cc3e838d2462b67f7d                   │
│  │ Version: 92487850                                                                        │
│  │ Digest: 76zC9EsJQfPTa9abjrvFdvCS5oqESyTrQNKSxZVb1HZu                                     │
│  └──                                                                                        │
│                                                                                             │
│ Transaction Kind: Programmable                                                              │
│   No input objects for this transaction                                                     │
│ ╭──────────────────────────────────────────────────────────────────────────────────╮        │
│ │ Commands                                                                         │        │
│ ├──────────────────────────────────────────────────────────────────────────────────┤        │
│ │ 0  MoveCall:                                                                     │        │
│ │  ┌                                                                               │        │
│ │  │ Function:  hello                                                              │        │
│ │  │ Module:    hello_world                                                        │        │
│ │  │ Package:   0x323bd5ddafa8e15c9aec735b2c34b28198caee69d6657fd0cb8021b88a150967 │        │
│ │  └                                                                               │        │
│ ╰──────────────────────────────────────────────────────────────────────────────────╯        │
│                                                                                             │
│ Signatures:                                                                                 │
│    1gUe/6pGtCNIIFxsoPpGrcz3UcfezJnQpe2hiA/GSplHcvE7Viehn3vi8mc0ZmBuh55ylTNhTH/EcGM63w7cCA== │
│                                                                                             │
╰─────────────────────────────────────────────────────────────────────────────────────────────╯
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Transaction Effects                                                                               │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Digest: EMqwhupyxNM7MBEfhLGDvTLmE8YHdzASEzoBrXhWKNJb                                              │
│ Status: Success                                                                                   │
│ Executed Epoch: 571                                                                               │
│ Mutated Objects:                                                                                  │
│  ┌──                                                                                              │
│  │ ID: 0x5085ad52a308634fbb2dc7cd25177475c8bc99b2c48804cc3e838d2462b67f7d                         │
│  │ Owner: Account Address ( 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767 )  │
│  │ Version: 92487851                                                                              │
│  │ Digest: DExvKHkHkouYMsXzRmWjawr4fLzKG1btDNmV3eLotWpa                                           │
│  └──                                                                                              │
│ Gas Object:                                                                                       │
│  ┌──                                                                                              │
│  │ ID: 0x5085ad52a308634fbb2dc7cd25177475c8bc99b2c48804cc3e838d2462b67f7d                         │
│  │ Owner: Account Address ( 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767 )  │
│  │ Version: 92487851                                                                              │
│  │ Digest: DExvKHkHkouYMsXzRmWjawr4fLzKG1btDNmV3eLotWpa                                           │
│  └──                                                                                              │
│ Gas Cost Summary:                                                                                 │
│    Storage Cost: 988000 MIST                                                                      │
│    Computation Cost: 1000000 MIST                                                                 │
│    Storage Rebate: 978120 MIST                                                                    │
│    Non-refundable Storage Fee: 9880 MIST                                                          │
│                                                                                                   │
│ Transaction Dependencies:                                                                         │
│    AYRm9Sbz1tBJXsVDRkFPNEgSjfMQwvJMNSpE7LXRUYDm                                                   │
│    DQBGth6dS6F9dP3ntbLJhN7sxJdVWoHrjDzSSNsH1vYN                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─────────────────────────────╮
│ No transaction block events │
╰─────────────────────────────╯

╭──────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Object Changes                                                                                   │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Mutated Objects:                                                                                 │
│  ┌──                                                                                             │
│  │ ObjectID: 0x5085ad52a308634fbb2dc7cd25177475c8bc99b2c48804cc3e838d2462b67f7d                  │
│  │ Sender: 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767                    │
│  │ Owner: Account Address ( 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767 ) │
│  │ ObjectType: 0x2::coin::Coin<0x2::sui::SUI>                                                    │
│  │ Version: 92487851                                                                             │
│  │ Digest: DExvKHkHkouYMsXzRmWjawr4fLzKG1btDNmV3eLotWpa                                          │
│  └──                                                                                             │
╰──────────────────────────────────────────────────────────────────────────────────────────────────╯
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Balance Changes                                                                                   │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│  ┌──                                                                                              │
│  │ Owner: Account Address ( 0x269ddb4ebfaeade29c5b357a0d92fdd169f195a0adc35fa779d50a5363b73767 )  │
│  │ CoinType: 0x2::sui::SUI                                                                        │
│  │ Amount: -1009880                                                                               │
│  └──                                                                                              │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
```

以上就是我们编写的第一个Hello World的Move程序。我们成功地将其发布在测试链上并调用了它。
