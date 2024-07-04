# ECVRF 可验证随机函数

可验证随机函数 (VRF) 是一种加密原语，使您能够生成随机数并提供该数字使用密钥生成的证据。任何人都可以使用与秘密密钥对应的公钥来验证证明，因此您可以将其用作随机数生成器 (RNG)，生成任何人都可以验证的输出。需要链上可验证随机性的应用程序也可以从其使用中受益。

## VRF 规范

Sui 中的 Move API 中使用的 VRF 是遵循 CFRG VRF 草案规范版本 15 的椭圆曲线 VRF (ECVRF)。它使用带有 SHA-512 哈希函数的 Ristretto255 椭圆曲线组构造。随机数是根据 RFC6979 生成的。

任何遵循与套件字符串 sui_vrf 相同规范的实现（请参阅 VRF 规范中的第 5 节）都可以用于计算 VRF 输出并生成证明。

fastcrypto 库为此类实现提供了 CLI 工具，并在以下示例中使用。

### 生成密钥​

从 fastcrypto 存储库的根目录中，运行以下命令来生成密钥对：

```plain
cargo run --bin ecvrf-cli keygen
```

这会输出十六进制格式的密钥和公钥。秘密密钥和公钥都是 32 字节字符串：

```plain
Secret key: c0cbc5bf0b2f992fe14fee0327463c7b03d14cbbcb38ce2584d95ee0c112b40b
Public key: 928744da5ffa614d65dd1d5659a8e9dd558e68f8565946ef3d54215d90cba015
```

### 计算 VRF 输出和证明​

要使用之前生成的密钥对计算输入字符串 `Hello, world!` （十六进制的 `48656c6c6f2c20776f726c6421` ）的 VRF 输出和证明，请运行以下命令：

```plain
cargo run --bin ecvrf-cli prove --input 48656c6c6f2c20776f726c6421 --secret-key c0cbc5bf0b2f992fe14fee0327463c7b03d14cbbcb38ce2584d95ee0c112b40b
```

这应该是 80 字节证明和 VRF 64 字节输出，均为十六进制格式：

```plain
Proof:  18ccf8bf316f00b387fc6e7b26f2d3ddadbf5e9c66d3a30986f12b208108551f9c6da87793a857d79261338a50430074b1dbc7f8f05e492149c51313381248b4229ebdda367146dbbbf95809c7fb330d
Output: 2b7e45821d80567761e8bb3fc519efe5ad80cdb4423227289f960319bbcf6eea1aef30c023617d73f589f98272b87563c6669f82b51dafbeb5b9cf3b17c73437
```

### 验证证明​

您可以使用 Sui Move 框架中的 sui::ecvrf::ecvrf_verify 验证智能合约中的证明和输出：

```move
module math::ecvrf_test {
    use sui::ecvrf;
    use sui::event;

    /// Event on whether the output is verified
    struct VerifiedEvent has copy, drop {
        is_verified: bool,
    }

    public fun verify_ecvrf_output(output: vector<u8>, alpha_string: vector<u8>, public_key: vector<u8>, proof: vector<u8>) {
        event::emit(VerifiedEvent {is_verified: ecvrf::ecvrf_verify(&output, &alpha_string, &public_key, &proof)});
    }
}
```

您还可以使用CLI工具进行验证：

```plain
cargo run --bin ecvrf-cli verify --output 2b7e45821d80567761e8bb3fc519efe5ad80cdb4423227289f960319bbcf6eea1aef30c023617d73f589f98272b87563c6669f82b51dafbeb5b9cf3b17c73437 --proof 18ccf8bf316f00b387fc6e7b26f2d3ddadbf5e9c66d3a30986f12b208108551f9c6da87793a857d79261338a50430074b1dbc7f8f05e492149c51313381248b4229ebdda367146dbbbf95809c7fb330d --input 48656c6c6f2c20776f726c6421 --public-key 928744da5ffa614d65dd1d5659a8e9dd558e68f8565946ef3d54215d90cba015
```

上面的命令返回验证：

```plain
Proof verified correctly!
```
