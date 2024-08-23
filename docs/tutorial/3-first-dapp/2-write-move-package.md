# 编写 Move 包

首先，在您计划存储包裹的位置打开终端或控制台。使用 bfc move new 命令创建一个名为 `my_first_package` 的空 Move 包：

```plain
$ bfc move new my_first_package
```

运行上一个命令会创建一个具有您提供的名称的目录（在本例中为 `my_first_package` ）。该命令使用包含 `sources` 目录和 `Move.toml` 清单文件的 Move 项目骨架填充新目录。使用文本编辑器打开清单以查看其内容：

```toml
# my_first_package/Move.toml
[package]
name = "my_first_package"
edition = "2024.alpha"
version = "0.0.1"
# license = ""           # e.g., "MIT", "GPL", "Apache 2.0"
# authors = ["..."]      # e.g., ["Joe Smith (joesmith@noemail.com)", "John Snow (johnsnow@noemail.com)"]

[dependencies]
Sui = { git = "https://github.com/benfenorg/bfc.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "v1.16.1" }

[addresses]
my_first_package = "0x0"
```

清单文件内容包括清单的可用部分以及提供附加信息的注释。在 Move 中，您可以在一行前面添加哈希标记 ( # ) 以表示注释。

- `[package]`：包含包的元数据。默认情况下， `bfc move new` 命令仅填充元数据的 name 值。在本例中，该示例将 `my_first_package` 传递给命令，该命令将成为包的名称。您可以删除 `[package]` 部分后续行的前 `#` 行，以为其他可用元数据字段提供值。
- `[dependencies]`：列出您的包运行所依赖的其他包。默认情况下， `bfc move new` 命令将 GitHub 上的 bfc 包列为单独依赖项。
- `[addresses]`：声明您的包使用的命名地址。默认情况下，该部分包含您使用 `bfc move new` 命令创建的包和 `0x0` 地址。发布过程将 `0x0` 地址替换为实际的链上地址。

## 定义包​

您现在有一个包，但它没有任何作用。为了使您的包有用，您必须添加定义模块的 .move 源文件中包含的逻辑。使用文本编辑器或命令行在包的 sources 目录中创建名为 my_module.move 的第一个包源文件：

```bash
$ touch my_first_package/sources/my_module.move
```

使用以下代码填充`my_module.move`文件：

```rust
module my_first_package::my_module {

    // Part 1: Import
    use sui::object::{Self, UID};
    use sui::transfer;
    use sui::tx_context::{Self, TxContext};

    // Part 2: struct definitions
    struct Sword has key, store {
        id: UID,
        magic: u64,
        strength: u64,
    }

    struct Forge has key {
        id: UID,
        swords_created: u64,
    }

    // Part 3: Module initializer to be executed when this module is published
    fun init(ctx: &mut TxContext) {
        let admin = Forge {
            id: object::new(ctx),
            swords_created: 0,
        };
        // Transfer the forge object to the module/package publisher
        let sender = tx_context::sender(ctx);
        transfer::transfer(admin, sender);
    }

    // Part 4: Accessors required to read the struct fields
    public fun magic(self: &Sword): u64 {
        self.magic
    }

    public fun strength(self: &Sword): u64 {
        self.strength
    }

    public fun swords_created(self: &Forge): u64 {
        self.swords_created
    }

    // Part 5: Public/entry functions (introduced later in the tutorial)

    // Part 6: Tests

}
```

前面代码中的注释突出显示了典型 Move 源文件的不同部分。

第 1 部分：导入 - 代码重用是现代编程的必要条件。 Move 通过 `use` 别名支持此概念，允许您的模块引用其他模块中声明的类型和函数。在此示例中，模块从 `object` 、 `transfer` 和 `tx_context` 模块导入，但不需要显式执行此操作，因为编译器提供了 `use` 语句。这些模块可供包使用，因为 `Move.toml` 文件定义了定义它们的依赖项（以及命名地址）。

第 2 部分：结构体声明 - 结构体定义模块可以创建或销毁的类型。结构定义可以包括由 `has` 关键字提供的功能。例如，本例中的结构具有 `key` 能力，这表明这些结构是可以在地址之间传输的 `object` 对象。结构体上的 `store` 能力提供了出现在其他结构体字段中并自由传输的能力。

第 3 部分：模块初始值设定项 - 模块发布时仅调用一次的特殊函数。

第 4 部分：访问器函数 - 这些函数允许从其他模块读取模块结构的字段。

保存文件后，您就拥有了完整的 Move 包。
