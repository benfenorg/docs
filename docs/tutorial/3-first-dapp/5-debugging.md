# 调试

Move 目前没有本机调试器。但是，您可以使用 · 模块将任意值打印到控制台。以这种方式监视变量值可以深入了解模块的逻辑。为此，首先在源文件中声明调试模块的别名，以便更简洁的访问：

```
use std::debug;
```

然后在您想要打印值 `v` 的地方，无论其类型如何，添加以下代码：

```
debug::print(&v);
```

如果 `v` 已经是引用，则为以下内容：

```
debug::print(v);
```

调试模块还提供了打印当前堆栈跟踪的函数：

```
debug::print_stack_trace();
```

或者，任何对中止或断言失败的调用也会打印失败时的堆栈跟踪。

# 在 my_module 中使用调试​

要查看正在运行的模块，请更新您的 `my_module` 代码以包含调试调用。具体来说，更新 `new_sword` 函数，以便在更新 `swords_created` 之前和之后打印 `forge` 的值。另外，包含 `print_stack_trace` 以使该函数如下所示：

```
public fun new_sword(
    forge: &mut Forge,
    magic: u64,
    strength: u64,
    ctx: &mut TxContext,
): Sword {
    debug::print(forge);
    forge.swords_created = forge.swords_created + 1;
    debug::print(forge);
    debug::print_stack_trace();
    Sword {
        id: object::new(ctx),
        magic: magic,
        strength: strength,
    }
}
```

要查看结果，请运行模块的测试。

```
$ bfc move test
```

当测试调用 `new_sword` 函数时，响应会打印出预期结果。

```
INCLUDING DEPENDENCY Bfc
INCLUDING DEPENDENCY MoveStdlib
BUILDING my_first_package
Running Move unit tests
[ PASS    ] 0x0::my_module::test_module_init
[debug] 0x0::my_module::Forge {
  id: 0x2::object::UID {
    id: 0x2::object::ID {
      bytes: @0x34401905bebdf8c04f3cd5f04f442a39372c8dc321c29edfb4f9cb30b23ab96
    }
  },
  swords_created: 0
}
[debug] 0x0::my_module::Forge {
  id: 0x2::object::UID {
    id: 0x2::object::ID {
      bytes: @0x34401905bebdf8c04f3cd5f04f442a39372c8dc321c29edfb4f9cb30b23ab96
    }
  },
  swords_created: 1
}
Call Stack:
    [0] 0000000000000000000000000000000000000000000000000000000000000000::my_module::test_module_init

        Code:
            [35] LdU64(7)
            [36] MutBorrowLoc(3)
            [37] Call(15)
          > [38] Call(5)
            [39] LdConst(0)
            [40] CallGeneric(2)
            [41] ImmBorrowLoc(3)

        Locals:
            [0] -
            [1] { { { <OBJECT-ID-WITHOUT-0x> } }, 1 }
            [2] -
            [3] { 2, { 00000000000000000000000000000000000000000000000000000000000000ad, [2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0], 0, 0, 0 } }


Operand Stack:

[ PASS    ] 0x0::my_module::test_sword_transactions
Test result: OK. Total tests: 2; passed: 2; failed: 0
```

输出显示增量后 `Forge` 更改的 `swords_created` 字段的值。堆栈跟踪显示到目前为止已执行的字节码指令以及接下来要执行的几条指令。

:::info
具体的字节码偏移量和局部变量的索引可能会根据 Bfc 工具链的版本而有所不同。
:::
