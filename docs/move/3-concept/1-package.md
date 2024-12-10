# 包

Move作为一种用于编写智能合约的语言，它实际上编译后被存储在区块链上。单个程序被组织成一个包（Package），包发布在区块链上并由一个唯一地址标识。可以通过发送PTB交易来与已发布的包进行交互，它还可以充当其他包的依赖项。

要创建一个新的包，请使用`bfc move new`命令。

一个包由若干个模块组成，一个模块又包含若干个函数、类型等。包的逻辑结构如下：

```plain
package 0x0
    module hello
        struct Hello
        fun hello_world()
    module pay
        struct Payment
        fun pay_to()
```

在本地开发环境中，包是一个目录，它包含`Move.toml`文件、`sources`目录、`tests`目录以及其他资源文件和目录。`Move.toml`文件被称为“包清单（Package Manifest）”，它包含有关包的元数据、源代码目录、测试代码目录等。一个包的目录结构通常看起来像这样：

```plain
sources/
    my_module.move
    another_module.move
    ...
tests/
    test_my_module.move
    test_another_module.move
    ...
docs/
    README.md
Move.toml
```

其中，仅`Move.toml`和`sources`目录是必需的，其他目录都是可选的。`tests`目录包含Move的测试代码，它在编译后并不会发布在链上，而是仅在测试中可用。

## 已发布的包

在开发过程中，包还暂时没有地址，因此设置为`0x0`。一旦发布包，它就会在区块链上获得一个唯一的地址，其中包含其模块的字节码。已发布的包是不可变的，用户或其他模块可以通过发送PTB事务与它进行交互。

一个已发布的包逻辑结构如下：

```plain
0x1a2b3c4d5e6f...
    my_module: <bytecode>
    another_module: <bytecode>
```
