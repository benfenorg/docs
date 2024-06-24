# 使用 TypeScript SDK 的客户端应用程序

此练习与本节前面主题中构建的示例不同。该指令不是向正在运行的示例添加前端，而是引导您在 React 应用程序中设置 dApp Kit，允许您连接到钱包，并从 Sui RPC 节点查询数据以显示在您的应用程序中。您可以使用它为之前使用的示例创建自己的前端，但如果您想要快速启动并运行功能齐全的应用程序，请在终端或控制台中运行以下命令，以使用本练习中的所有步骤构建新应用程序已经实施：

:::info
您必须使用pnpm或yarn包管理器来创建Sui项目脚手架。如果需要，请按照 pnpm install 或yarn install 说明进行操作。
:::

```
pnpm create @mysten/dapp --template react-client-dapp
```

或者：

```
yarn create @mysten/dapp --template react-client-dapp
```

### Sui TypeScript SDK

Sui TypeScript SDK (@mysten/sui) 提供了通过 TypeScript 与 Sui 生态系统交互所需的所有低级功能。您可以在任何 TypeScript 或 JavaScript 项目中使用它，包括 Web 应用程序、Node.js 应用程序或使用支持 TypeScript 的 React Native 等工具编写的移动应用程序。

### dApp Kit

dApp Kit (@mysten/dapp-kit) 是 React hooks、组件和实用程序的集合，使在 Sui 上构建 dApp 变得简单。有关 dApp 套件的更多信息，请参阅 dApp 套件文档。

### 安装依赖项​

首先，您需要一个 React 应用程序。以下步骤适用于任何 React，因此您可以按照相同的步骤将 dApp Kit 添加到现有 React 应用程序。如果您要开始一个新项目，您可以使用 Vite 搭建一个新的 React 应用程序。

在终端或控制台中运行以下命令，选择 React 作为框架，然后选择 TypeScript 模板之一：

```
npm init vite
```

现在您已经有了 React 应用程序，您可以安装必要的依赖项来使用 dApp Kit：

```
npm install @mysten/sui @mysten/dapp-kit @tanstack/react-query
```

### 设置Provider组件​

要使用 dApp Kit 的所有功能，请使用几个 Provider 组件包装您的应用程序。

打开渲染应用程序的根组件（Vite 模板使用的默认位置是 `src/main.tsx` ）并使用以下代码集成或替换当前代码。

第一个要设置的 `Provider` 是 `@tanstack/react-query` 中的 `QueryClientProvider` 。这个 `Provider` 管理 dApp 套件中各种钩子的请求状态。如果您已经在使用 `@tanstack/react-query` ，dApp Kit 可以共享相同的 `QueryClient` 实例。

```
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();

ReactDOM.createRoot(document.getElementById('root')!).render(
	<React.StrictMode>
		<QueryClientProvider client={queryClient}>
			<App />
		</QueryClientProvider>
	</React.StrictMode>,
);
```

接下来，设置 `SuiClientProvider` 。此 `Provider` 将 `SuiClient` 实例从 `@mysten/sui` 传递到 dApp Kit 中的所有钩子。该提供商管理 dApp Kit 连接的网络，并且可以接受多个网络的配置。此练习连接到 `devnet` 。

```
import { SuiClientProvider } from '@mysten/dapp-kit';
import { getFullnodeUrl } from '@mysten/sui/client';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();
const networks = {
	devnet: { url: getFullnodeUrl('devnet') },
	mainnet: { url: getFullnodeUrl('mainnet') },
};

ReactDOM.createRoot(document.getElementById('root')!).render(
	<React.StrictMode>
		<QueryClientProvider client={queryClient}>
			<SuiClientProvider networks={networks} defaultNetwork="devnet">
				<App />
			</SuiClientProvider>
		</QueryClientProvider>
	</React.StrictMode>,
);
```

最后，从 `@mysten/dapp-kit` 设置 `WalletProvider` ，并导入 `dapp-kit` 组件的样式。

```
import '@mysten/dapp-kit/dist/index.css';

import { SuiClientProvider, WalletProvider } from '@mysten/dapp-kit';
import { getFullnodeUrl } from '@mysten/sui/client';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();
const networks = {
	devnet: { url: getFullnodeUrl('devnet') },
	mainnet: { url: getFullnodeUrl('mainnet') },
};

ReactDOM.createRoot(document.getElementById('root')!).render(
	<React.StrictMode>
		<QueryClientProvider client={queryClient}>
			<SuiClientProvider networks={networks} defaultNetwork="devnet">
				<WalletProvider>
					<App />
				</WalletProvider>
			</SuiClientProvider>
		</QueryClientProvider>
	</React.StrictMode>,
);
```

### 连接到钱包​

设置完所有 `Providers` 后，您就可以使用 dApp Kit 挂钩和组件了。要允许用户将他们的钱包连接到您的 dApp，请添加 `ConnectButton` 。

```
import { ConnectButton } from '@mysten/dapp-kit';

function App() {
	return (
		<div className="App">
			<header className="App-header">
				<ConnectButton />
			</header>
		</div>
	);
}
```

`ConnectButton` 组件显示一个按钮，单击该按钮即可打开模式，使用户能够连接其钱包。连接后，它会显示他们的地址，并提供断开连接的选项。

### 获取连接的钱包地址​

现在您已经有了让用户连接钱包的方法，您可以开始使用 `useCurrentAccount` 挂钩来获取有关已连接钱包帐户的详细信息。

```
import { ConnectButton, useCurrentAccount } from '@mysten/dapp-kit';

function App() {
	return (
		<div className="App">
			<header className="App-header">
				<ConnectButton />
			</header>

			<ConnectedAccount />
		</div>
	);
}

function ConnectedAccount() {
	const account = useCurrentAccount();

	if (!account) {
		return null;
	}

	return <div>Connected to {account.address}</div>;
}
```

### 从Sui RPC节点查询数据​

现在您已经拥有要连接的帐户，您可以查询连接的帐户拥有的对象：

```
import { useCurrentAccount, useSuiClientQuery } from '@mysten/dapp-kit';

function ConnectedAccount() {
	const account = useCurrentAccount();

	if (!account) {
		return null;
	}

	return (
		<div>
			<div>Connected to {account.address}</div>;
			<OwnedObjects address={account.address} />
		</div>
	);
}

function OwnedObjects({ address }: { address: string }) {
	const { data } = useSuiClientQuery('getOwnedObjects', {
		owner: address,
	});
	if (!data) {
		return null;
	}

	return (
		<ul>
			{data.data.map((object) => (
				<li key={object.data?.objectId}>
					<a href={`https://example-explorer.com/object/${object.data?.objectId}`}>
						{object.data?.objectId}
					</a>
				</li>
			))}
		</ul>
	);
}
```

您现在有一个连接到钱包的 dApp，并且可以从 RPC 节点查询数据。
