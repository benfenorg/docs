# 发布包

在调用 Move 包中的函数（超出模拟的 Bfc 执行场景）之前，该包必须在 Bfc 网络上可用。当你发布一个包时，你实际上是在网络上创建一个任何人都可以访问的不可变的 Bfc 对象。

要将包发布到 Bfc 网络，请在包的根目录中使用 `publish` CLI 命令。

```
$ bfc client publish
```

如果发布交易成功，您的终端或控制台会响应发布交易的详细信息，这些详细信息分为几个部分，包括交易数据、交易效果、交易块事件、对象更改和余额更改。

在“对象更改”表中，您可以在“已发布的对象”部分找到有关刚刚发布的包的信息。您的响应具有实际的 `PackageID` ，它以 `0x123...ABC` 形式标识包（而不是 `<PACKAGE-ID>` ）。

```
╭─────────────────────────────────────────────────────────────────────╮
│ Object Changes                                                      │
├─────────────────────────────────────────────────────────────────────┤
│ Created Objects:                                                    │
│  ...                                                                │
|                                                                     |
│ Mutated Objects:                                                    │
│  ...                                                                │
|                                                                     |
│ Published Objects:                                                  │
│  ┌──                                                                │
│  │ PackageID: <PACKAGE-ID>                                          │
│  │ Version: 1                                                       │
│  │ Digest: <DIGEST-HASH>                                            │
│  │ Modules: my_module                                               │
│  └──                                                                │
╰─────────────────────────────────────────────────────────────────────╯
```

您当前的活动地址现在拥有三个对象（或更多，如果您在此示例之前有对象）。假设您使用新地址，运行 `bfc objects` 命令会显示这些对象是什么。

```
$ bfc client objects

╭───────────────────────────────────────────────────────────────────────────────────────╮
│ ╭────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ objectId   │  <OBJECT-ID>                                                         │ │
│ │ version    │  10                                                                  │ │
│ │ digest     │  <DIGEST-HASH>                                                       │ │
│ │ objectType │  <PACKAGE-ID>::my_module::Forge                                      │ │
│ ╰────────────┴──────────────────────────────────────────────────────────────────────╯ │
│ ╭────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ objectId   │  <OBJECT-ID>                                                         │ │
│ │ version    │  10                                                                  │ │
│ │ digest     │  <DIGEST-HASH>                                                       │ │
│ │ objectType │  0x0000..0002::coin::Coin                                            │ │
│ ╰────────────┴──────────────────────────────────────────────────────────────────────╯ │
│ ╭────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ objectId   │  <OBJECT-ID>                                                         │ │
│ │ version    │  10                                                                  │ │
│ │ digest     │  <DIGEST-HASH>                                                       │ │
│ │ objectType │  0x0000..0002::package::UpgradeCap                                   │ │
│ ╰────────────┴──────────────────────────────────────────────────────────────────────╯ │
╰───────────────────────────────────────────────────────────────────────────────────────╯
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

事务执行后，您可以再次使用 `bfc client objects` 命令检查 `Sword` 对象的状态。如果您使用您的地址作为 `<TO-ADDRESS>` ，您现在应该总共看到四个对象：

```
╭───────────────────────────────────────────────────────────────────────────────────────╮
│ ╭────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ objectId   │  <OBJECT-ID>                                                         │ │
│ │ version    │  11                                                                  │ │
│ │ digest     │  <DIGEST-HASH>                                                       │ │
│ │ objectType │  <PACKAGE-ID>::my_module::Forge                                      │ │
│ ╰────────────┴──────────────────────────────────────────────────────────────────────╯ │
│ ╭────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ objectId   │  <OBJECT-ID>                                                         │ │
│ │ version    │  11                                                                  │ │
│ │ digest     │  <DIGEST-HASH>                                                       │ │
│ │ objectType │  0x0000..0002::coin::Coin                                            │ │
│ ╰────────────┴──────────────────────────────────────────────────────────────────────╯ │
│ ╭────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ objectId   │  <OBJECT-ID>                                                         │ │
│ │ version    │  11                                                                  │ │
│ │ digest     │  <DIGEST-HASH>                                                       │ │
│ │ objectType │  <PACKAGE-ID>::my_module::Sword                                      │ │
│ ╰────────────┴──────────────────────────────────────────────────────────────────────╯ │
│ ╭────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ objectId   │  <OBJECT-ID>                                                         │ │
│ │ version    │  10                                                                  │ │
│ │ digest     │  <DIGEST-HASH>                                                       │ │
│ │ objectType │  0x0000..0002::package::UpgradeCap                                   │ │
│ ╰────────────┴──────────────────────────────────────────────────────────────────────╯ │
╰───────────────────────────────────────────────────────────────────────────────────────╯
```

恭喜！您已成功将包发布到 Bfc 网络，并使用可编程交易块修改了区块链状态。
