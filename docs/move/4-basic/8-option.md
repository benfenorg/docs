# Option

大多数编程语言使用`null`表示空对象，然而null会带来很多潜在的问题。Move没有null值，要表示空，必须使用Option。

Option是Move标准库定义的一种类型，表示可能存在也可能不存在的可选值。Move的Option是从Rust借鉴的，它是Move中非常有用的原语。Option定义如下：

```
// move-stdlib/source/option.move
struct Option<Element> has copy, drop, store {
    vec: vector<Element>
}
```

模块`std::option`会隐式导入，不需要使用import导入。

Option是一个通用类型，它采用类型参数Element。它有一个单一的字段vec，向量的长度可以为1或0，用于表示值的存在或不存在。

Option类型有两种变体：Some和None。Some变体包含值，None变体表示不存在值。Option类型用于以类型安全的方式表示值的缺失，避免需要null值。

为了展示为什么Option类型是必要的，让我们看一个例子。考虑一个应用程序，它接受用户输入并将其存储在变量中。有些字段是必需的，有些是可选的。例如，用户的中间名是可选的。虽然我们可以使用空字符串来表示缺少中间名，但需要额外的检查来区分空字符串和缺少的中间名。相反，我们可以使用Option类型来表示中间名。

```
public struct User has drop {
    first_name: String,
    middle_name: Option<String>,
    last_name: String,
}

public fun register(
    first_name: String,
    middle_name: Option<String>,
    last_name: String,
): User {
    User { first_name, middle_name, last_name }
}
```

在上面的示例中，`middle_name`字段的类型为`Option<String>`。这意味着`middle_name`字段可以包含`String`值或为空。这清楚地表明中间名是可选的，并且避免了需要进行额外检查来区分空字符串和缺失的中间名。

## 使用Option

要使用Option类型，可以使用`some()`或`none()`方法创建`Option`：

```
// 创建带值的Option:
let mut opt = option::some(b"Alice");

// 使用 is_some() 判断是否有值:
assert!(opt.is_some() == true);

// 使用 borrow() / borrow_mut() 来借用值:
assert!(opt.borrow() == &b"Alice");

// 使用 extract() 获得值并释放, 调用后Option将为空:
let inner = opt.extract();
assert!(opt.is_none() == true);
```
