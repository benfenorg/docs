# Move 的链上签名验证

Bfc 中的 Move 合约支持链上多种签名方案的验证。并非所有链上验证支持的签名都支持作为用户签名验证。有关交易授权的有效签名方案，请参阅 Bfc Signatures。

本主题涵盖：

1. 如何使用 fastcrypto 的 CLI 工具创建给定方案的签名。仅用于测试和调试，请勿在生产中使用。
2. 通过提交签名、消息和公钥，调用链上的 Move 方法进行验证。

签名方案涵盖：

- Ed25519签名（64字节）
- Secp256k1 不可恢复签名（64 字节）
- Secp256k1 可恢复签名（65 字节）
- Secp256r1 不可恢复签名（64 字节）
- Secp256r1 可恢复签名（65 字节）
- BLS G1 签名（minSig 设置）
- BLS G2 签名（minPk 设置）

## 用途

### 设置 fastcrypto CLI 二进制文件​

```plain
git@github.com:MystenLabs/fastcrypto.git
cd fastcrypto/
cargo build --bin sigs-cli
```

## 使用 CLI 签名并提交到链上 Move 方法​

### Ed25519签名（64字节）​

1. 生成密钥并签署消息。

```plain
target/debug/sigs-cli keygen --scheme ed25519 --seed 0000000000000000000000000000000000000000000000000000000000000000                
Private key in hex: $SK
Public key in hex: $PK

target/debug/sigs-cli sign --scheme ed25519 --msg $MSG --secret-key  $SK

Signature in hex: $SIG
Public key in hex: $PK
```

2. 调用Move中的`verify`方法。所有输入均以十六进制格式的字节表示：

```move
use sui::ed25519;

let msg = x"$MSG";
let pk = x"$PK";
let sig = x"$SIG";
let verify = ed25519::ed25519_verify(&sig, &pk, &msg);
assert!(verify == true, 0);
```

### Secp256k1不可恢复签名（64字节）​

1. 生成密钥并签署消息。

```plaintarget/debug/sigs-cli keygen --scheme secp256k1 --seed 0000000000000000000000000000000000000000000000000000000000000000                
Private key in hex: $SK
Public key in hex: $PK

target/debug/sigs-cli sign --scheme secp256k1 --msg $MSG --secret-key $SK

Signature in hex: $SIG
Public key in hex: $PK
```

2. 调用Move中的`verify`方法。所有输入均以十六进制格式的字节表示：

```move
use sui::ecdsa_k1;

let msg = x"$MSG";
let pk = x"$PK";
let sig = x"$SIG";
// The last param 1 represents the hash function used is SHA256, the default hash function used when signing in CLI.
let verify = ecdsa_k1::secp256k1_verify(&sig, &pk, &msg, 1);
assert!(verify == true, 0);
```

### Secp256k1可恢复签名（65字节）​

1. 生成密钥并签署消息。

```plaintarget/debug/sigs-cli keygen --scheme secp256k1-rec --seed 0000000000000000000000000000000000000000000000000000000000000000                
Private key in hex: $SK
Public key in hex: $PK

target/debug/sigs-cli sign --scheme secp256k1-rec --msg $MSG --secret-key $SK

Signature in hex: $SIG
Public key in hex: $PK
```

2. 调用Move中的`ecrecover`方法。所有输入均以十六进制格式的字节表示：

```move
use sui::ecdsa_k1;

let msg = x"$MSG";
let pk = x"$PK";
let sig = x"$SIG";
// The last param 1 represents the hash function used is SHA256, the default hash function used when signing in CLI.
let recovered = ecdsa_k1::secp256k1_ecrecover(&sig, &msg, 1);
assert!(pk == recovered, 0);
```

### Secp256r1不可恢复签名（64字节）​

1. 生成密钥并签署消息。

```plain
target/debug/sigs-cli keygen --scheme secp256r1 --seed 0000000000000000000000000000000000000000000000000000000000000000                
Private key in hex: $SK
Public key in hex: $PK

target/debug/sigs-cli sign --scheme secp256r1 --msg $MSG --secret-key $SK

Signature in hex: $SIG
Public key in hex: $PK
```

2. 调用Move中的`verify`方法。所有输入均以十六进制格式的字节表示：

```move
use sui::ecdsa_r1;

let msg = x"$MSG";
let pk = x"$PK";
let sig = x"$SIG";
// The last param 1 represents the hash function used is SHA256, the default hash function used when signing in CLI.
let verify = ecdsa_r1::secp256r1_verify(&sig, &pk, &msg, 1);
assert!(verify == true, 0);
```

### Secp256r1可恢复签名（65字节）​

1. 生成密钥并签署消息。

```plain
target/debug/sigs-cli keygen --scheme secp256r1-rec --seed 0000000000000000000000000000000000000000000000000000000000000000                
Private key in hex: $SK
Public key in hex: $PK

target/debug/sigs-cli sign --scheme secp256r1-rec --msg $MSG --secret-key $SK

Signature in hex: $SIG
Public key in hex: $PK
```

2. 调用Move中的`ecrecover`方法。所有输入均以十六进制格式的字节表示：

```move
use sui::ecdsa_r1;

let msg = x"$MSG";
let pk = x"$PK";
let sig = x"$SIG";
// The last param 1 represents the hash function used is SHA256, the default hash function used when signing in CLI.
let recovered = ecdsa_r1::secp256r1_ecrecover(&sig, &msg, 1);
assert!(pk == recovered, 0);
```

### BLS G1签名（48字节，minSig设置）​

1. 生成密钥并签署消息。

```plain
target/debug/sigs-cli keygen --scheme bls12381-minsig --seed 0000000000000000000000000000000000000000000000000000000000000000                
Private key in hex: $SK
Public key in hex: $PK

target/debug/sigs-cli sign --scheme bls12381-minsig --msg $MSG --secret-key $SK

Signature in hex: $SIG
Public key in hex: $PK
```

2. 调用Move中的`verify`方法。所有输入均以十六进制格式的字节表示：

```move
use sui::bls12381;

let msg = x"$MSG";
let pk = x"$PK";
let sig = x"$SIG";
let verified = bls12381::bls12381_min_sig_verify(&sig, &pk, &msg);
assert!(verified == true, 0);
```

### BLS G1签名（96字节，minPk设置）​

1. 生成密钥并签署消息。

```plain
target/debug/sigs-cli keygen --scheme bls12381-minpk --seed 0000000000000000000000000000000000000000000000000000000000000000                
Private key in hex: $SK
Public key in hex: $PK

target/debug/sigs-cli sign --scheme bls12381-minpk --msg $MSG --secret-key $SK

Signature in hex: $SIG
Public key in hex: $PK
```

2. 调用Move中的verify方法。所有输入均以十六进制格式的字节表示：

```move
use sui::bls12381;

let msg = x"$MSG";
let pk = x"$PK";
let sig = x"$SIG";
let verified = bls12381::bls12381_min_pk_verify(&sig, &pk, &msg);
assert!(verified == true, 0);
```
