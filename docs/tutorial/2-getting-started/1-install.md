# 安装 BenFen

安装 BenFen 的最快方法是使用每个版本附带的二进制文件。如果您需要对安装过程进行更多控制，可以从源安装。要利用容器化，您可以使用我们提供的 Docker 映像。

## 支持的操作系统​

BenFen 支持以下操作系统：

- Linux - Ubuntu 22.04
- macOS
- Windows

## 二进制安装

每个 BenFen 的发行版都提供了一组适用于多个操作系统的二进制文件。您可以从 GitHub 下载这些二进制文件并使用它们来安装 BenFen。

访问 https://github.com/benfenlab/benfen ，在右侧面板找到 `Release`，点击下载最新版本。

## 从源代码构建

要从源码开始构建 BenFen，您需要首先搭建编译环境。

### 安装 Rust

如果您正在使用 Linux 或 macOS，要下载 Rustup 并安装 Rust，请在终端中运行以下命令，然后遵循屏幕上的指示：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

如果您正在使用 Windows，请下载并运行 [rustup-init.exe](https://static.rust-lang.org/rustup/dist/i686-pc-windows-gnu/rustup-init.exe)。

### 安装依赖包

如果您正在使用 Linux，我们假定您使用的是基于 APT 包管理器的发行版，可以使用以下命令一次性安装这些依赖包：

```bash
sudo apt-get install curl git-all cmake gcc libssl-dev pkg-config libclang-dev libpq-dev build-essential
```
