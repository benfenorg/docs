# String

Move语言本身没有内置的字符串类型，但是标准库中有两个标准的字符串实现：`std::string`模块定义了`UTF-8`编码的String类型和方法，`std::ascii`提供了ASCII编码的String类型及其方法。

通常情况下，使用`std::string`模块就足够了，因为ASCII编码是UTF-8编码的一部分。

在Move中，最重要的是理解String其实就是若干个字节，本质上，String是`vector<u8>`的包装器：

```
public struct String has copy, drop, store {
    bytes: vector<u8>,
}
```

因为String仅仅是`vector<u8>`的包装器，因此，String和`vector<u8>`可以通过以下方法互相转换：

```
let v = b"Hello"; // vector<u8>
let s = v.to_string(); // String
let v2 = v.as_bytes(); // &vector<u8>
```

上述String和`vector<u8>`的转换并不检查字符有效性。如果要检查是否是有效的UTF-8字符，可以使用`try_to_string()`：

```
let v1 = b"Hello";
let opt1 = v1.try_to_string(); // Option<String>
assert!(opt1.is_some());

let v2: vector<u8> = vector[0, 1, 2];
let opt2 = v2.try_to_string();
assert!(opt2.is_none());
```

以`try_`开头的函数或方法会返回`Option`，这是借鉴Rust语言的一种通用方式。

## 常用操作

String提供了许多处理字符串的方法，最常用的操作是连接、切片和获取长度：

```
let mut str = b"Hello,".to_string();
let another = b" World!".to_string();

str.append(another); // str现在是"Hello World!"

let sub = str.sub_string(0, 5); // "Hello"

str.length(); // 12
str.is_empty(); // false
```

注意，无论是`sub_string()`还是`length()`，均是针对字节操作。因为UTF-8是一种变长编码，String也不提供按字符访问的方法。
