# 准备工作

您使用 Move 编写存在于链上的包，这意味着它们存在于您发布到的 BenFen 网络上。本节中的说明将引导您完成编写基本包、调试和测试代码以及​​发布的过程。您需要遵循这些说明才能完成练习。

您可以使用 move BenFen CLI 命令来获取一些说明。 BenFen CLI 随二进制文件一起安装，因此如果按照安装说明进行操作，您的系统上就会有它。要验证您是否已安装它，请在终端或控制台中运行以下命令。

```bash
$ bfc --version
```

如果控制台没有响应类似以下的版本号，请参阅安装 BenFen 的说明。

## 连接到网络​

安装 BenFen 后，您可以连接到网络。 BenFen 拥有两个公共网络（Testnet、Mainnet），您还可以运行并连接到本地 BenFen 网络。对于每个网络，您需要一个特定于该网络的链上地址。地址是一个具有唯一 ID 的对象，格式为 `BFC8bd4613c004aac53d06bb7ceb7f46832c9ae69bdc105dfc5fcac225d2061fcac` 。除了该地址之外，您还需要 BFC 支付与链上活动相关的 Gas 费，例如发布包以及对这些包进行 Move 调用。对于除主网之外的所有网络，您的帐户都可以获得免费的 BFC 币，以方便软件包开发。出于本示例的目的，请连接到 Testnet 网络。

## 连接到测试网​

如果您已经设置了网络配置，请将活动环境切换到测试网。以下说明适用于初始设置。

在终端或控制台中，使用以下命令开始配置：

```plain
$ bfc client
```

在提示符处，键入 `y` 并按 Enter 连接到 BenFen Full 节点服务器。

在出现以下提示时，键入测试网服务器的地址 `https://testrpc.benfen.org:443` （注：请勿省略443端口）并按 Enter 。

在出现以下提示时，键入 `testnet` 为网络指定别名，然后按 Enter 。您可以在后续命令中使用别名，而不是键入完整的 URL。

在出现以下提示时，键入 `0` 并按 Enter 。该选择将在 ed25519 签名方案中创建一个地址。

```plain
Generated new keypair for address with scheme "ed25519" [BFCeaf74459ede83446e5fea66d8fbf9e91c5988a6ba9d2d7321209dfdab968e894]
Secret Recovery Phrase : [found corn simple ... useful middle repeat]
Client for interacting with the Bfc network
```

响应显示了您的地址和秘密恢复短语，请务必保存此信息以供以后参考。由于这是在测试网网络上，因此该信息的安全性并不像在主网那样重要。

在您的终端或控制台中，使用以下命令为您的帐户获取 BFC。

```plain
$ bfc client faucet
```

您可以使用以下命令确认您收到了 BFC。根据网络的活动，接收硬币可能会出现延迟。

```plain
$ bfc client gas
                               Object ID                                | Gas Value
-------------------------------------------------------------------------------------
BFCa4e233f6d41ba5d0e7f9240172c903adff415fced7d9f818d885e4024d16fa60f5ad | 5000000000
BFCccf4ad77ba86bf6dd262d28ef59727b043fadb56805620eb8f8a638f17f3109fed8f | 15000000000
```

您现在已连接到 BenFen Testnet 网络，并且应该拥有一个具有可用 BFC 的帐户。
