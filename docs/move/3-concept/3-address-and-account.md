# 地址和账户

## 地址

地址是区块链上的唯一标识符，它用于识别包、帐户和对象。地址的固定大小为32字节，通常表示为前缀为`0x`的十六进制字符串。

地址不区分大小写，以下是一个有效的地址：

```plain
0x21a81c3e51ff04d71de4f76e22b9e312b68c7c670ddf990872e29b045cd2d6f1
```

上面的地址长度为64个字符（32个字节），并以0x为前缀。

如果地址长度不足64个字符，则以`0`填充。地址`0x123`与以下地址等同：

```plain
0x0000000000000000000000000000000000000000000000000000000000000123
```

BenFen保留了用于标准包和对象的地址。保留地址通常是易于记忆和输入的简单值。以下是部分保留地址：

- 0x1：标准库的地址，别名`std`；
- 0x2：Sui框架的地址，别名`sui`；
- 0x6：系统Clock对象的地址；
- 0x8：系统随机数对象的地址；
- 0x403：系统`DenyList`对象的地址。

## 帐户

帐户是识别用户的一种方式，一个用户可以拥有多个账户。

一个账户由私钥生成，并由地址标识。帐户可以拥有对象，并可以发送交易。每笔交易都有一个发送者，发送者由地址标识。

BenFen支持多种密码算法来生成账户。目前账户支持的算法是ed25519和secp256k1。还有一种特殊的生成账户的方式为zklogin。
