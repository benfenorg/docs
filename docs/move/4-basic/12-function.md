# 函数

函数是Move语言最核心的可重用代码块。函数以fun关键字在模块中声明，默认情况下，函数是私有的，仅本模块的其他函数可以访问：

```
module example::hello;

fun sqr(x: u64): u64 {
    x * x
}

#[test]
fun test_sqr() {
    assert!(sqr(0) == 0);
    assert!(sqr(5) == 25);
    assert!(sqr(123) == 15129);
}
```

在上面的例子中，我们定义了一个函数`sqr`，它具有一个u64参数，返回值为u64。同时，我们还定义了一个`test_sqr`函数，它以`#[test]`标识，这表示这是一个测试函数，它可以在本地运行，但正式发布时不会被编译，也就不会被部署至链上。

函数名约定以小写字母+下划线构成，这与C函数的命名一致。默认情况下，函数的最后一个表达式就是函数的返回值，但也可以用return语句在函数内部任意处提前返回：

```
fun find_max(a: u64, b: u64): u64 {
    if (a > b) {
        return a
    }
    b
}
```

## 调用函数

对于一个函数来说，完整的名称是`包名::模块名::函数名`。同一个模块内，可以只写函数名，但要调用的函数在不同的模块，则需要以use语句导入：

```
module example::hello;

use string::String;

public fun register(name: vector<u8>) {
    // 调用String模块的utf8函数:
    let str = String::utf8(name);

    // 调用String模块的length函数:
    let len = String::length(&str);
}
```

在Move函数中，如果函数的第一个参数是一个结构体，那么，可以将函数调用改写为类似面向对象的`对象.方法()`的调用：

```
module example::hello;

use string::String;

public fun register(name: vector<u8>) {
    // utf8(b: vector<u8>)
    let str = name.utf8();

    // length(s: &String):
    let len = str.length();
}
```

这种方式简化了函数调用。我们在编写结构体时，通常会定义一系列操作该结构体的函数，将第一个参数定义为结构体本身或其引用，可以将函数视为结构体的方法，能简化调用。

## 返回多个值

一个函数可以返回多个值，返回多个值实际上是返回一个元组（Tuple）：

```
fun max_and_min(vs: vector<u64>): (u64, u64) {
    let mut max = vs.index_of(0);
    let mut min = vs.index_of(1);
    for (let i=1; i<vs.length(); i++) {
        let x = vs.index_of(i);
        if (max < x) {
            max = x;
        }
        if (min > x) {
            min = x;
        }
    }
    (max, min)
}
```

当一个函数返回多个值时，调用者必须用解包的方式将多个值依次放到对应的变量中去：

```
let vs: vector<u64> = vector[100, 34, 699, 21, 999, 77];
let (a, b) = max_and_min(vs);
```

如果要忽略某些返回值，可以用占位符`_`：

```
// 仅需要max, 不需要min:
let (a, _) = max_and_min(vs);
```

## 修饰符

默认情况下，函数仅模块内部可见。要让外部模块可以访问一个函数，需要用`public`修饰：

```
module example::hello;

public fun max(a: u64, b: u64): u64 {
    ...
}
```

要仅允许同一个包的外部模块访问，需要用`public(package)`修饰：

```
module example::hello;

public(package) fun max(a: u64, b: u64): u64 {
    ...
}
```

由于上述函数使用了`public(package)`修饰，函数本身位于`example::hello`中，则`example::test`可以访问它，因为它们具有相同的包`example`。`another::hello`则无法访问它，因为它们位于不同的包。
