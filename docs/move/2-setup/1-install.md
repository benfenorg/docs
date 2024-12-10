# 安装命令行环境

Move是一种编译语言，因此您需要安装编译器才能编写和运行Move程序。该编译器包含在 BenFen 二进制文件中，可以使用以下方法安装。

## 下载命令行文件

您可以从[GitHub](https://github.com/benfenorg/bfc/releases)下载最新的 BenFen 命令行可执行文件。该可执行文件适用于macOS和Linux。

## 编译BenFen

可以在本地编译和构建BenFen，需要首先安装Rust。

使用如下命令在系统中安装最新的Rust：

```plain
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
$ rustup update stable
```

从[GitHub](https://github.com/benfenorg/bfc)下载源码并根据README文件编译BenFen。
