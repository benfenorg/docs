# 创建 NFT

在 Bfc 上创建 NFT 与其他非基于对象的区块链不同。这些区块链需要专用标准来处理定义 NFT 的属性，因为它们基于智能合约和代币 ID 之间的映射。例如，以太坊上的 ERC-721 标准需要将全球唯一 ID 与相关智能合约地址配对，以在网络上创建唯一的代币实例。

在 Bfc 上，每个对象都已经有一个唯一的 ID，因此无论您是处理一百万个可互换代币（如硬币），还是处理数千个具有个体特征的 NFT（如 SuiFrens），您在 Bfc 上的智能合约始终与单个对象进行交互。

想象一下，您在 Bfc 和另一个不基于对象的区块链上创建了一个 Excitable Chimp NFT 集合。要在另一个区块链上获取诸如黑猩猩名称之类的属性，您需要与创建 NFT 的智能合约进行交互，以使用 NFT ID 获取该信息（通常来自链外存储）。在 Bfc 上，名称属性可以是定义 NFT 本身的对象上的字段。这种构造提供了一个更直接的过程来访问 NFT 的元数据，因为需要信息的智能合约只需从对象本身返回名称即可。

### 示例

以下示例在 Bfc 上创建一个基本的 NFT。 `TestnetNFT` 结构体使用 `id` 、 `name` 、 `description` 和 `url` 字段定义 NFT。

```
// examples/move/nft/sources/testnet_nft.move

public struct TestnetNFT has key, store {
    id: UID,
    name: string::String,
    description: string::String,
    url: Url,
}
```

在这个例子中，任何人都可以通过调用 `mint_to_sender` 函数来铸造 NFT。顾名思义，该函数创建一个新的 `TestnetNFT` 并将其传输到进行调用的地址。

```
// examples/move/nft/sources/testnet_nft.move

#[allow(lint(self_transfer))]
public fun mint_to_sender(
    name: vector<u8>,
    description: vector<u8>,
    url: vector<u8>,
    ctx: &mut TxContext
) {
    let sender = ctx.sender();
    let nft = TestnetNFT {
        id: object::new(ctx),
        name: string::utf8(name),
        description: string::utf8(description),
        url: url::new_unsafe_from_bytes(url)
    };

    event::emit(NFTMinted {
        object_id: object::id(&nft),
        creator: sender,
        name: nft.name,
    });

    transfer::public_transfer(nft, sender);
}
```

该模块还包括返回 NFT 元数据的函数。参考之前使用的假设，您可以调用 name 函数来获取该值。正如你所看到的，该函数只是返回 NFT 本身的名称字段值。

```
// examples/move/nft/sources/testnet_nft.move

public fun name(nft: &TestnetNFT): &string::String {
    &nft.name
}
```
