# 包清单

在一个包中，`Move.toml`是一个描述包及其依赖项的清单文件。它以TOML格式编写，包含多个部分，其中最重要的是`[package]`、`[dependencies]`和`[addresses]`。

```plain
# Move.toml
[package]
name = "hello_world"
version = "0.0.1"
edition = "2024"

[dependencies]
Sui = { git = "https://github.com/benfenorg/bfc.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "framework/testnet" }

[addresses]
std = "0x1"
alice = "0xA11CE"
```

## Package描述

`[package]`用于描述包，它不影响在链上发布，但用于工具和发布管理：

- name：包的名称；
- version：包的版本，可用于发布管理；
- edition：Move语言版本，目前有效的值是`2024`。

## Dependencies描述

`[dependencies]`用于描述当前包的依赖项，每个依赖项都被指定为一个键值对，其中键是依赖项的名称，值是依赖项的地址，地址可以是本地文件，也可以是远程Git仓库的URL路径。

### Git仓库

以Git仓库指定依赖项时，请注明Git仓库的地址、子目录路径与分支名称：

```plain
Sui = { git = "https://github.com/benfenorg/bfc.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "framework/testnet" }
```

### 本地目录

以本地目录指定依赖项时，请注明本地目录的路径：

```plain
MyPackage = { local = "../path/to/my-package" }
```

### 自动依赖

某个依赖的包还可以依赖其他包，例如，Sui依赖MoveStdlib，因此，我们当前编写的包也自动依赖MoveStdlib，并可以直接在代码中使用。

### 版本冲突

有时，同一个包的依赖项存在冲突版本，这时，可以用`override=true`表示存在冲突时，将使用`[dependencies]`指定的依赖项版本，而不是依赖项本身指定的版本：

```plain
[dependencies]
Pay = { override = true, git = "https://...", subdir = "...", rev = "1.0" }
Swap = { git = "https://...", subdir = "...", rev = "2.0" }
```

假设`Pay`包依赖`Swap 1.0`，因为设置了`override=true`，实际依赖的`Swap`包的版本为我们当前指定的`2.0`。

### 开发模式的依赖

可以将`[dev-dependencies]`添加到清单中，它可以覆盖开发和测试中的依赖关系。例如，想要在dev模式下使用不同版本的包，可以添加自定义的依赖规范`[dev-dependencies]`部分：

```plain
[dependencies]
Pay = { git = "https://...", subdir = "...", rev = "2.0" }

[dev-dependencies]
Pay = { git = "https://...", subdir = "...", rev = "2.0" }
```

## 地址别名

`[addresses]`用于添加地址的别名，可以对任何地址指定一个别名，然后在代码中使用别名。

例如，如果在此部分添加`alice = "0x1a2b3c"`，则可以在代码中将`alice`用作`0x1a2b3c`。

### 开发模式的地址别名

`[dev-addresses]`与`[addresses]`类似，但此处定义的别名仅用于测试和开发模式。需要注意的是，在`[dev-addresses]`中不能引入新的别名，只能覆盖现有的别名。因此，在上面的示例中，可以设置`alice = "0x4d5e6f"`，但无法新定义`bob = "0x12345"`。定义后`alice`地址在测试和开发模式中将为`0x4d5e6f`，在常规构建中为`0x1a2b3c`。
