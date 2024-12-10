# 模块

模块是组织Move代码的基本单位。模块用于分组和隔离代码，默认情况下，模块的所有成员都是模块私有的。在本节中，您将学习如何定义模块，如何声明其成员，以及如何从其他模块访问它们。

## 模块声明

一个模块就是一个Move文件，模块使用`module`关键字声明，后跟包地址、模块名称、分号和模块主体。模块名称应采用`snake_case`：使用小写字母、单词之间用下划线分隔。模块名称在一个包中必须是唯一的。

通常，`sources`文件夹中的单个`.move`文件包含单个模块。文件名应与模块名称匹配，例如，`coffee_shop`模块应存储在`coffee_shop.move`文件中。

以下是一个`book_shop`模块，它位于包`book`中：

```rust
// 声明模块:
module book::book_shop;

// 模块代码...
```

还有一种旧的模块声明方法，即用`{}`将模块的代码括起来：

```rust
// 声明模块:
module book::book_shop {
    // 模块代码...
}
```

使用这种块声明时，可以在一个`.move`文件中声明多个模块。不过，大多数情况下，推荐使用第一种方式声明模块。

## 导入其他模块

在模块声明后，通常会导入该模块需要用到的其他模块或对象，例如：

```rust
use std::string; // 导入标准库模块
use sui::url::URL; // 导入URL对象
use book::payment; // 导入另一个自定义模块
```

## 模块代码

模块代码通常包括结构体、常量以及函数。以下是示例代码：

```rust
// 结构体/struct:
public struct BookShop { ... }

// 常量/constant:
const PRICE: u32 = 100;

// 函数/function:
entry fun mint_to_sender(name: vector<u8>, url: vector<u8>, ctx: &mut TxContext) { ... }
```
