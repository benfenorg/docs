# 签署和发送交易

Sui 中的交易表示对特定功能的调用（例如调用智能合约函数），这些功能在输入上执行以定义交易的结果。

输入可以是对象引用（对拥有的对象、不可变对象或共享对象），也可以是编码值（例如，用作 Move 调用的参数的字节向量）。交易构建后，通常通过使用可编程交易块（PTB），用户对交易进行签名并提交到链上执行。

签名由钱包拥有的私钥提供，其公钥必须与交易发送者的 Sui 地址一致。

Sui 使用 `SuiKeyPair` 生成签名，该签名提交到意图消息的 `Blake2b` 哈希摘要 ( `intent || bcs bytes of tx_data` )。目前支持的签名方案有 `Ed25519 Pure` 、 `ECDSA Secp256k1` 、 `ECDSA Secp256r1` 、 `Multisig` 和 `zkLogin` 。

您可以使用 `SuiKeyPair` 实例化 `Ed25519 Pure` 、 `ECDSA Secp256k1` 和 `ECDSA Secp256r1` 并使用它来签署交易。请注意，本指南不适用于 `Multisig` 和 `zkLogin`。

有了签名和交易字节，就可以提交交易并执行。

###  工作流程​

以下高级流程描述了构建、签署和执行链上交易的整体工作流程：

- 通过创建链接多个交易的 `Transaction` 来构建交易数据。有关更多信息，请参阅构建可编程事务块。
- SDK 内置的 Gas 估算和代币选择会选择 Gas 代币。
- 对交易进行签名以生成签名。
- 提交 `Transaction` 及其签名以供链上执行。

:::info
如果您想使用特定的gas币，首先找到用于支付gas的gas币对象ID，并在PTB中明确使用它。如果没有gas币对象，则使用splitCoin交易创建gas币对象。分币交易应该是PTB中的第一个交易调用。
:::

### 示例

以下示例演示了如何使用 Rust、TypeScript 或 Sui CLI 签署和执行交易。

有多种方法可以实例化密钥对并使用 Sui TypeScript SDK 派生其公钥和 Sui 地址。

```
import { fromHEX } from '@mysten/bcs';
import { getFullnodeUrl, SuiClient } from '@mysten/sui/client';
import { type Keypair } from '@mysten/sui/cryptography';
import { Ed25519Keypair } from '@mysten/sui/keypairs/ed25519';
import { Secp256k1Keypair } from '@mysten/sui/keypairs/secp256k1';
import { Secp256r1Keypair } from '@mysten/sui/keypairs/secp256r1';
import { Transaction } from '@mysten/sui/transactions';

const kp_rand_0 = new Ed25519Keypair();
const kp_rand_1 = new Secp256k1Keypair();
const kp_rand_2 = new Secp256r1Keypair();

const kp_import_0 = Ed25519Keypair.fromSecretKey(
	fromHex('0xd463e11c7915945e86ac2b72d88b8190cfad8ff7b48e7eb892c275a5cf0a3e82'),
);
const kp_import_1 = Secp256k1Keypair.fromSecretKey(
	fromHex('0xd463e11c7915945e86ac2b72d88b8190cfad8ff7b48e7eb892c275a5cf0a3e82'),
);
const kp_import_2 = Secp256r1Keypair.fromSecretKey(
	fromHex('0xd463e11c7915945e86ac2b72d88b8190cfad8ff7b48e7eb892c275a5cf0a3e82'),
);

// $MNEMONICS refers to 12/15/18/21/24 words from the wordlist, e.g. "retire skin goose will hurry this field stadium drastic label husband venture cruel toe wire". Refer to [Keys and Addresses](/concepts/cryptography/transaction-auth/keys-addresses.mdx) for more.
const kp_derive_0 = Ed25519Keypair.deriveKeypair('$MNEMONICS');
const kp_derive_1 = Secp256k1Keypair.deriveKeypair('$MNEMONICS');
const kp_derive_2 = Secp256r1Keypair.deriveKeypair('$MNEMONICS');

const kp_derive_with_path_0 = Ed25519Keypair.deriveKeypair('$MNEMONICS', "m/44'/784'/1'/0'/0'");
const kp_derive_with_path_1 = Secp256k1Keypair.deriveKeypair('$MNEMONICS', "m/54'/784'/1'/0/0");
const kp_derive_with_path_2 = Secp256r1Keypair.deriveKeypair('$MNEMONICS', "m/74'/784'/1'/0/0");

// replace `kp_rand_0` with the variable names above.
const pk = kp_rand_0.getPublicKey();
const sender = pk.toSuiAddress();

// create an example transaction block.
const txb = new Transaction();
txb.setSender(sender);
txb.setGasPrice(5);
txb.setGasBudget(100);
const bytes = await txb.build();
const serializedSignature = (await keypair.signTransaction(bytes)).signature;

// verify the signature locally
expect(await keypair.getPublicKey().verifyTransaction(bytes, serializedSignature)).toEqual(true);

// define sui client for the desired network.
const client = new SuiClient({ url: getFullnodeUrl('testnet') });

// execute transaction.
let res = client.executeTransactionBlock({
	transactionBlock: bytes,
	signature: serializedSignature,
});
console.log(res);
```
