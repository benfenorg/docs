# Groth16

零知识证明允许证明者验证陈述是否正确，而无需透露有关输入的任何信息。例如，证明者可以验证他们是否知道数独谜题的解决方案，而无需透露解决方案。

零知识简洁非交互式知识论证（zk-SNARK）是一系列非交互式零知识证明，具有简洁的证明大小和高效的验证时间。其中一个重要且广泛使用的变体是基于配对的 zk-SNARK，例如 Groth16 证明系统，它是最有效且使用最广泛的系统之一。

Sui 中的 Move API 使您能够在 BN254 或 BLS12-381 椭圆曲线结构上使用 Groth16 zk-SNARK 有效地验证可以以 NP 完全语言表达的任何语句。

有一些高级语言可以表达这些语句，例如下面示例中使用的 Circcom。

Groth16 需要每个电路都有可信设置来生成验证密钥。该 API 不固定任何特定的验证密钥，每个用户都可以生成自己的参数或对其应用程序使用现有的验证。

## 用途

以下示例演示了如何根据 Circom 编写的语句创建 Groth16 证明，然后使用 Sui Move API 进行验证。该 API 目前支持最多八个公共输入。

###  创建电路​

该证明表明我们知道散列函数的秘密输入，该散列函数给出了特定的公共输出。

```circom
pragma circom 2.1.5;

include "node_modules/circomlib/circuits/poseidon.circom";

template Main() {
    component poseidon = Poseidon(1);
    signal input in;
    signal output digest;
    poseidon.inputs[0] <== in;
    digest <== poseidon.out;
}

component main = Main();
```

我们使用 Poseidon 哈希函数，它是 ZK 友好的哈希函数。假设已经安装了circom编译器，则使用以下命令编译上述电路：

```plain
circom main.circom --r1cs --wasm
```

这将输出 R1CS 格式的约束和 Wasm 格式的电路。

### 生成证明​

要生成可在 Sui 中验证的证明，您需要生成一个见证人。此示例使用 Arkworks 的 ark-circom Rust 库。该代码为电路构建了一个见证，并针对给定的输入生成了一个证明。最后验证证明的正确性。

```move
use ark_bn254::Bn254;
use ark_circom::CircomBuilder;
use ark_circom::CircomConfig;
use ark_groth16::Groth16;
use ark_snark::SNARK;

fn main() {
    // Load the WASM and R1CS for witness and proof generation
    let cfg = CircomConfig::<Bn254>::new("main.wasm", "main.r1cs").unwrap();

    // Insert our secret inputs as key value pairs. We insert a single input, namely the input to the hash function.
    let mut builder = CircomBuilder::new(cfg);
    builder.push_input("in", 7);

    // Create an empty instance for setting it up
    let circom = builder.setup();

    // WARNING: The code below is just for debugging, and should instead use a verification key generated from a trusted setup.
    // See for example https://docs.circom.io/getting-started/proving-circuits/#powers-of-tau.
    let mut rng = rand::thread_rng();
    let params =
        Groth16::<Bn254>::generate_random_parameters_with_reduction(circom, &mut rng).unwrap();

    let circom = builder.build().unwrap();

    // There's only one public input, namely the hash digest.
    let inputs = circom.get_public_inputs().unwrap();

    // Generate the proof
    let proof = Groth16::<Bn254>::prove(&params, circom, &mut rng).unwrap();

    // Check that the proof is valid
    let pvk = Groth16::<Bn254>::process_vk(&params.vk).unwrap();
    let verified = Groth16::<Bn254>::verify_with_processed_vk(&pvk, &inputs, &proof).unwrap();
    assert!(verified);
}
```

该证明表明，输入 (7) 在使用 `Poseidon` 哈希函数进行哈希处理时会给出一定的输出（在本例中为 `inputs[0].to_string()` = `7061949393491957813657776856458368574501817871421526214197139795307327923534` ）。

## 验证

BFC 中用于验证证明的 API 需要一个特殊处理的验证密钥，其中仅使用值的子集。理想情况下，每个电路只对这个准备好的验证密钥进行一次计算。您可以使用 BFC Move API 的 sui::groth16::prepare_verifying_key 方法以及先前使用的 params.vk 值的序列化来执行此处理。

`prepare_verifying_key` 函数的输出是一个具有四个字节数组的向量，分别对应于 `vk_gamma_abc_g1_bytes` 、 `alpha_g1_beta_g2_bytes` 、 `gamma_g2_neg_pc_bytes` 、 `delta_g2_neg_pc_bytes` 。

要验证证明，您还需要另外两个输入 `public_inputs_bytes` 和 `proof_points_bytes` ，它们分别包含公共输入和证明。这些是上一个示例中 `inputs` 和 `proof` 值的序列化，您可以在 Rust 中计算如下：

```rust
let mut vk_bytes = Vec::new();
params.vk.serialize_compressed(&mut vk_bytes).unwrap();

let mut public_inputs_bytes = Vec::new();
for i in 0..inputs.len() { // if there is more than one public input, serialize one by one
    inputs[i].serialize_compressed(&mut inputs_bytes).unwrap();
}

let mut proof_points_bytes = Vec::new();
proof.serialize_compressed(&mut proof_points_bytes).unwrap();
```

以下示例智能合约准备验证密钥并验证相应的证明。此示例使用 `BN254` 椭圆曲线构造，该构造作为 `prepare_verifying_key` 和 `verify_groth16_proof` 函数的第一个参数给出。您可以使用 `bls12381` 函数来代替 `BLS12-381` 构造。

```move
module test::groth16_test {
    use sui::groth16;
    use sui::event;

    /// Event on whether the proof is verified
    struct VerifiedEvent has copy, drop {
        is_verified: bool,
    }

    public fun verify_proof(vk_bytes: vector<u8>, public_inputs_bytes: vector<u8>, proof_points_bytes: vector<u8>) {
        let pvk = groth16::prepare_verifying_key(&groth16::bn254(), &vk_bytes);
        let public_inputs = groth16::public_proof_inputs_from_bytes(public_inputs_bytes);
        let proof_points = groth16::proof_points_from_bytes(proof_points_bytes);
        event::emit(VerifiedEvent {is_verified: groth16::verify_groth16_proof(&groth16::bn254(), &pvk, &public_inputs, &proof_points)});
    }
}
```
