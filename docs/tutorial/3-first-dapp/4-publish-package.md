# 发布包

在调用 Move 包中的函数（超出模拟的 Bfc 执行场景）之前，该包必须在 Bfc 网络上可用。当你发布一个包时，你实际上是在网络上创建一个任何人都可以访问的不可变的 Bfc 对象。

要将包发布到 Bfc 网络，请在包的根目录中使用 `publish` CLI 命令。

```plain
$ bfc client publish --gas-budget 10000000
```

如果发布交易成功，您的终端或控制台会响应发布交易的详细信息，这些详细信息分为几个部分，包括交易数据、交易效果、交易块事件、对象更改和余额更改。

在“对象更改”表中，您可以在“已发布的对象”部分找到有关刚刚发布的包的信息。您的响应具有实际的 `PackageID` ，它以 `BFC123...ABC` 形式标识包（而不是 `<PACKAGE-ID>` ）。

```plain
----- Object changes ----
Array [
    Object {
        "type": String("mutated"),
        "sender": String("BFCeaf74459ede83446e5fea66d8fbf9e91c5988a6ba9d2d7321209dfdab968e8949a28"),
        "owner": Object {
            "AddressOwner": String("BFCeaf74459ede83446e5fea66d8fbf9e91c5988a6ba9d2d7321209dfdab968e8949a28"),
        },
        "objectType": String("0x2::coin::Coin<0x2::bfc::BFC>"),
        "objectId": String("BFCa4e233f6d41ba5d0e7f9240172c903adff415fced7d9f818d885e4024d16fa60f5ad"),
        "version": String("7"),
        "previousVersion": String("6"),
        "digest": String("BvkDf1kJduTK1zeyrmcRcNLJa33tHFekreF81GP56vN"),
    },
    Object {
        "type": String("published"),
        "packageId": String("BFC08d22a8798e2ffe7b65d3d3f0d193969033a238ac3b661589369b627793337bb654f"),
        "version": String("1"),
        "digest": String("D3qjU15bGjn74zDVajKyuqN4dendRxbpDX8DRmAL1x1P"),
        "modules": Array [
            String("example"),
        ],
    },
    Object {
        "type": String("created"),
        "sender": String("BFCeaf74459ede83446e5fea66d8fbf9e91c5988a6ba9d2d7321209dfdab968e8949a28"),
        "owner": Object {
            "AddressOwner": String("BFCeaf74459ede83446e5fea66d8fbf9e91c5988a6ba9d2d7321209dfdab968e8949a28"),
        },
        "objectType": String("0x2::package::UpgradeCap"),
        "objectId": String("BFC5fdb7eb75c8e419d445ec4342ff1519d33e798cdb8ea1f9fcb84f97d7963f8f90de8"),
        "version": String("7"),
        "digest": String("2DqFadYiHEjCfjtNispPW7gv6TjKLQQeoNjamCKcaVeo"),
    },
    Object {
        "type": String("created"),
        "sender": String("BFCeaf74459ede83446e5fea66d8fbf9e91c5988a6ba9d2d7321209dfdab968e8949a28"),
        "owner": Object {
            "AddressOwner": String("BFCeaf74459ede83446e5fea66d8fbf9e91c5988a6ba9d2d7321209dfdab968e8949a28"),
        },
        "objectType": String("0x08d22a8798e2ffe7b65d3d3f0d193969033a238ac3b661589369b627793337bb::example::Forge"),
        "objectId": String("BFCc2623d749c99f3ca966ecf2b0d8f266de3e9be1e588a410636fb7972a4b648f009a1"),
        "version": String("7"),
        "digest": String("3quSV4MH4aN2RiJYDwKioVYiR1wd4959eqktrQQYWqXP"),
    },
]
```

您当前的活动地址现在拥有三个对象（或更多，如果您在此示例之前有对象）。假设您使用新地址，运行 `bfc objects` 命令会显示这些对象是什么。

```plain
$ bfc client objects
                 Object ID                  |  Version   |                    Digest                    |   Owner Type    |               Object Type               
---------------------------------------------------------------------------------------------------------------------------------------------------------------------
 BFC5fdb7eb75c8e419d445ec4342ff1519d33e798cdb8ea1f9fcb84f97d7963f8f90de8 |     7      | EiU28q00mdX5Pw+CZ4Mdd04R2JXHsqf1jsQhQx4acdg= |  AddressOwner   | Some(Struct(MoveObjectType(Other(StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("package"), name: Identifier("UpgradeCap"), type_params: [] }))))
 BFCa4e233f6d41ba5d0e7f9240172c903adff415fced7d9f818d885e4024d16fa60f5ad |     7      | AsyT8knO1Nxy3o2U8yOG8BYdypul6ZHXH29+BOaZ/6s= |  AddressOwner   | Some(Struct(MoveObjectType(GasCoin(Struct(StructTag { address: 0000000000000000000000000000000000000000000000000000000000000002, module: Identifier("bfc"), name: Identifier("BFC"), type_params: [] })))))
 BFCc2623d749c99f3ca966ecf2b0d8f266de3e9be1e588a410636fb7972a4b648f009a1 |     7      | Kj6Ly9DgshyukpNuYtgSsM2B1un6W2fjk3zMbviARbo= |  AddressOwner   | Some(Struct(MoveObjectType(Other(StructTag { address: 08d22a8798e2ffe7b65d3d3f0d193969033a238ac3b661589369b627793337bb, module: Identifier("example"), name: Identifier("Forge"), type_params: [] }))))
Showing 3 results.
```

`objectId` 字段是每个对象的唯一标识符。

- `Coin` 对象：您从测试网水龙头收到了 `Coin` 对象。由于发布交易的天然气成本，它的价值比您收到它时略低。
- `Forge` 对象：回想一下， `init` 函数在包发布时运行。此示例包的 `init` 函数创建一个 `Forge` 对象并将其传输给发布者（您）。
- `UpgradeCap` 对象：您发布的每个包都会导致收到 `UpgradeCap` 对象。您可以使用此对象稍后升级软件包或将其销毁以使软件包无法升级。

### 与包交互​

现在包已经上链了，你可以调用它的函数来与包交互。您可以使用 `bfc client call` 命令对包函数进行单独调用，也可以使用 `bfc client ptb` 命令构造更高级的事务块。命令的 `ptb` 部分代表可编程事务块。简单来说，PTB 允许您将命令分组到单个事务中，以实现更高效、更具成本效益的网络活动。

例如，可以通过调用 `my_module` 包中的 `new_sword` 函数来创建包中定义的新 `Sword` 对象，然后传递 `Sword` ：

```
$ bfc client ptb \
	--assign forge @<FORGE-ID> \
	--assign to_address @<TO-ADDRESS> \
	--move-call <PACKAGE-ID>::my_module::new_sword forge 3 3 \
	--assign sword \
	--transfer-objects "[sword]" to_address \
	--gas-budget 20000000
```

:::info
您可以通过在地址和对象 ID 前面加上“@”前缀来传递它们。在某些情况下，需要将十六进制值与地址区分开。

对于本地钱包中的地址，您可以使用其别名（传递时不带“@”，例如 --transfer-objects my_alias）。
:::

确保将 `<FORGE-ID>` 、 `<TO-ADDRESS>` 和 `<PACKAGE-ID>` 替换为 `Forge` 对象的实际 `objectId` ，分别是收件人的地址（在本例中为您的地址）和包的 `packageID` 。

事务执行后，您可以再次使用 `bfc client objects` 命令检查 `Sword` 对象的状态。如果您使用您的地址作为 `<TO-ADDRESS>` ，您现在应该总共看到四个对象。

恭喜！您已成功将包发布到 Bfc 网络，并使用可编程交易块修改了区块链状态。
