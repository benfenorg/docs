# 中断执行

在Move语言中，当执行一个交易时，整个交易要么成功，要么失败。成功时，执行的所有操作——即对对象的修改会被记录到链上，而失败时，对象状态不会被修改，因此，交易具有类似SQL数据库的事务特性。

在SQL中，如果想要中止执行事务并回滚，可以通过rollback语句完成。类似的，在Move中，可以用abort语句中止交易的执行，中止后对象状态不会被修改。

一个常见的用例是检测到用户没有足够的余额时，调用abort并返回一个`u64`类型的中止代码：

```
if (! has_enough_balance) {
    abort 1
}
```

上述代码在检测到用户没有足够余额时，将中止并返回代码`1`。

也可以写`abort(1)`，这两种写法是等价的。

## 使用assert!

在Move中，还可以使用内置的`assert!`。`assert!`是一个内置的宏，可以直接作为断言使用：

```
assert!(has_enough_balance, 1);
```

它展开后实际上相当于以下语句：

```
if (!has_enough_balance) {
    abort 1
};
```

使用abort或assert!时，最好定义常量来返回中止代码：

```
const EInvalidArgument: u64 = 1;
const ENoAccess: u64 = 403;
const ENotFound: u64 = 404;

public fun swap(price: u32, to: address) {
    assert!(price > 0, EInvalidArgument);
    assert!(whitelist(address), ENoAccess);
    ...
}
```

## 错误消息

以u64表示的错误常量仍然不太直观，Move引入了一种特殊的错误常量，以`#[error]`标记，它允许将错误常量的类型定义为`vector<u8>`，实际上就是字符串类型常量：

```
#[error]
const EInvalidArgument = b"Invalid Argument";

#[error]
const ENoAccess = b"Access Denied";
```

使用`assert!`时，与使用`u64`常量一致：

```
#[error]
const EInvalidArgument = b"Invalid Argument";

#[error]
const ENoAccess = b"Access Denied";

public fun swap(price: u32, to: address) {
    assert!(price > 0, EInvalidArgument);
    assert!(whitelist(address), ENoAccess);
    ...
}
```
