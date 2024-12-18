# 能力

在上一节中，我们已经通过`struct`来定义结构体。默认情况下，一个结构体实例被创建后，必须解包后才能通过编译，这是没有`drop`能力的结构体的默认行为。

## 能力

在定义结构体时，可以同时声明能力。能力定义了该结构体能做出什么行为，能力通过has语法声明。Move一共定义了4中能力：

- drop：丢弃能力，表示该结构体可以被直接丢弃；
- copy：复制能力，表示该结构体允许被复制；
- key：对象能力，表示该结构体可以被视为对象；
- store：存储能力，表示该结构体可以被存储在对象中。

## drop

在普通股的编程语言中，对象创建后被丢弃，是一种默认行为。但是，在Move中，没有drop能力的结构体是不允许被直接丢弃的，必须以解包的方式处理它们。

如果一个结构体允许被丢弃，可以添加drop能力：

```
public struct Coffee has drop {
    name: String,
    price: u8,
};

#[test]
fun test() {
    let c1 = Coffee {
        name: b'Latte'.to_string(),
        price: 10
    };
    // 可以直接丢弃
}
```

## copy

在普通的编程语言中，对象默认就可以被复制。但是，Move是智能合约编程语言，默认情况下，一个结构体不能被复制。

要使得结构体可以被复制，必须添加copy能力：

```
public struct Coffee has copy {
    name: String,
    price: u8,
};

#[test]
fun test() {
    let mut c1 = Coffee {
        name: b'Latte'.to_string(),
        price: 10
    };

    let mut c2 = c1; // 此处发生了复制

    // c2和c1是不同的对象
    c2.name = b'Mocha'.to_string();
}
```

## key

具有key能力的结构体在Move中被认为是对象，一个对象可以被存储，并被一个账户所拥有。

定义key能力的结构体时，要求第一个字段必须以`id`命名，且类型为`UID`：

```
public struct Book has key {
    id: UID,
    name: String,
    price: u8
};
```

后续我们会更详细地讨论key能力。

## store

当一个对象——即具有key能力的结构体，除第一个id字段外，其他所有字段的类型必须具有store能力。

换句话说，对象是具有key能力的结构体，而具有store能力的结构体才能作为对象的字段：

```
public struct Config has store {
    suger: u8,
    ice: bool,
};

public struct Coffee has key {
    id: UID,
    name: String, // String具有store能力
    config: Config, // Config具有store能力
    price: u8, // u8也具有store能力
};
```

实际上，基本数据类型`bool`和`u8`~`u256`，以及`vector`、`address`类型均具有store能力，所以它们才能被用于对象的字段中。

