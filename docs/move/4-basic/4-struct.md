# 结构体

Move语言允许通过struct关键字定义结构体，这与C的结构体类似：

```
public struct Author has drop, copy {
    name: String,
    birth: u16,
}

public struct Book has drop, copy {
    author: Author,
    name: String,
    publish: u16,
    edition: Option<u8>,
}
```

默认情况下，结构体本身是私有的，因此，只能在其定义的模块内使用。结构体的字段也是私有的，模块外部无法修改，必须通过模块的公开函数实现修改。

注意到关键字`has`定义了结构体的能力，这里是`drop`和`copy`，后续我们会详细介绍，这里先加上，以防止后续代码无法编译通过。

## 创建结构体实例

定义了一个结构体后，我们可以初始化一个结构体并赋值给变量：

```
#[test_only]
module hello::hello_tests;

usd std::debug::print;

#[test]
fun test_hello() {
    let mut author = Author {
        name: b"Bob".to_string(),
        birth: 1990,
    };
    // 现在变量author是一个Author类型的结构体

    let mut book = Book {
        author: author,
        name: b"Move Programming Language".to_string(),
        publish: 2024,
    };
    // 现在book是一个Book类型的结构体

    print(&book.author.name); // Bob
    print(&author.name); // Bob

    // 修改book.author.name:
    book.author.name = b"Alice".to_string();

    print(&book.author.name); // Alice
    print(&author.name); // Bob
}
```

注意观察，当我们修改`book.author.name`时，并不影响`author`变量的`name`，原因是，创建`Book`实例时，传入的`author`具有`copy`能力，因此，`book`持有的`author`实际上是一个复制，而不是原始实例。

## 解包结构体

您可能注意到了，如果一个结构体没有声明任何能力，使用它后，编译器会发出错误：

```
public struct Coffee {
    name: String,
    price: u8
};

#[test]
fun test_hello() {
    let coffee = Coffee {
        name: b"Latte".to_string(),
        price: 10,
    };
}
```

上述代码编译器会报错：

```
The type 'Coffee' does not have the ability 'drop'.
```

这是因为结构体作为一个对象，不可在创建后丢弃。没有drop能力的结构体必须被存储或解包（Unpacking）。解包意味着该结构体被“拆解”为零散的字段：

```
#[test]
fun test_hello() {
    let coffee = Coffee {
        name: b"Latte".to_string(),
        price: 10,
    };
    // 解包:
    let Coffee { name, price } = coffee;
    // 正常编译通过
}
```

在Move中，创建一个结构体意味着创建了一个对象。Move不允许创建的对象不经处理就直接丢弃，这个特性强迫智能合约的开发者仔细地处理每一个创建的对象，避免类似Coin这样的对象创建后凭空消失，这是Move有别于其他编程语言的一个重要区别。

## 操作结构体

可以定义一系列函数来操作结构体。例如，定义两个函数来获取和设置Coffee价格：

```
public fun get_price(c: &Coffee): u8 {
    c.price
}

public fun set_price(c: &mut Coffee, p: u8) {
    c.price = p;
}
```

类似`&Coffee`这样的引用我们会在后面详细讨论。这里我们可以直接调用函数：

```
let mut coffee = Coffee { ... };
let p = get_price(&coffee);
set_price(&coffee, 20);
```

为了简化函数调用，在Move中，如果第一个参数是结构体本身的引用，那么，可以将函数调用转换为对结构体的方法调用：

```
let mut coffee = Coffee { ... };
let p = coffee.get_price();
coffee.set_price(20);
```

这两种调用方式是等价的，后一种调用更简单而且更直观。通常，编写一个结构体的同时，会提供若干这样的相关函数，这些函数可视为结构体的方法，这有助于模块化代码，并以更简单的方式编写代码。
