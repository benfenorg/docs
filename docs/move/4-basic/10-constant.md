# 常量

常量是定义为不可变的值，在Move语言中，常量用const定义在模块中：

```
module example::hello;

const DEFAULT_PRICE: u32 = 99;
const ADMIN_ADDR: address = @0xa1b2c3d4e5f6;
```

需要注意的是，在Move中，常量必须以大写字母开头。习惯上，常量总是以下划线分隔的大写字母表示。但是，对于表示错误的常量，通常的写法是`EAbcXyz`：

```
const PRICE: u32 = 100; // 普通常量
const EItemNotFound: u64 = 123; // 错误常量
```

常量被定义后，是不可变的。常量被编译器编译后存储在模块的字节码中，每次使用常量时，会复制该值。

## 使用配置

常量是模块私有的，在模块外部无法访问。但是，在大型DApp中，一个合约可能会被分拆为好几个模块。为了让所有模块都能访问常量，可以定义一个`config`模块，专门用于定义常量，并通过`public`函数来访问它们：

```
module example::config;

const DEFAULT_PRICE: u32 = 99;
const DEFAULT_TAX: u8 = 10;

public fun default_price(): u32 {
    DEFAULT_PRICE
}

public fun default_tax(): u8 {
    DEFAULT_TAX
}
```

这样，其他模块就可以通过`public`函数访问并使用常量。
