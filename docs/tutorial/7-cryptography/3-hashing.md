# 哈希

加密哈希函数是一种广泛使用的加密原语，它将任意长度的输入映射到固定长度的输出（即哈希值）。哈希函数被设计为单向函数，这意味着不可能通过逆函数从给定的哈希值中找到输入数据，并且具有抗碰撞性，这意味着不可能找到两个不同的数据映射到相同哈希值的输入。

Bfc Move API 支持以下加密哈希函数：

- SHA2-256 为 `std::hash::sha2_256`
- SHA3​​-256 为 `std::hash::sha3_256`
- Keccak256 为 `bfc::hash::keccak256`
- Blake2b-256 为 `bfc::hash::blake2b256`

## 用途

std::hash 模块的 Move 标准库中提供了 SHA2-256 和 SHA3-256 哈希函数。以下示例展示了如何在智能合约中使用 SHA2-256 哈希函数：

```move
module test::hashing_std {
    use std::hash;
    use sui::object::{Self, UID};
    use sui::tx_context::TxContext;
    use sui::transfer;
    use std::vector;

    /// Object that holds the output hash value.
    struct Output has key, store {
        id: UID,
        value: vector<u8>
    }

    public fun hash_data(data: vector<u8>, recipient: address, ctx: &mut TxContext) {
        let hashed = Output {
            id: object::new(ctx),
            value: hash::sha2_256(data),
        };
        // Transfer an output data object holding the hashed data to the recipient.
        transfer::public_transfer(hashed, recipient)
    }
}
```

Keccak256 和 Blake2b-256 哈希函数可通过 Bfc Move 库中的 bfc::hash 模块获得。下面显示了如何在智能合约中使用 Keccak256 哈希函数的示例。请注意，此处给出了哈希函数的输入作为参考。 Keccak256 和 Blake2b-256 都是这种情况。

```move
module test::hashing_sui {
    use sui::hash;
    use sui::object::{Self, UID};
    use sui::tx_context::TxContext;
    use sui::transfer;
    use std::vector;

    /// Object that holds the output hash value.
    struct Output has key, store {
        id: UID,
        value: vector<u8>
    }

    public fun hash_data(data: vector<u8>, recipient: address, ctx: &mut TxContext) {
        let hashed = Output {
            id: object::new(ctx),
            value: hash::keccak256(&data),
        };
        // Transfer an output data object holding the hashed data to the recipient.
        transfer::public_transfer(hashed, recipient)
    }
}
```
