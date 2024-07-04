# 自定义索引器

您可以使用 Sui 微数据摄取框架构建自定义索引器。要创建索引器，您需要订阅具有完整检查点内容的检查点流。该流可以是 Mysten Labs 公开可用的流之一、您在本地环境中设置的流，也可以是两者的组合。

建立自定义索引器有助于改善延迟，允许修剪 Sui Full 节点的数据，并提供检查点数据的高效组合。

## 接口及数据格式​

要使用该框架，请实现一个基本接口：

```rust
#[async_trait]
trait Worker: Send + Sync {
    async fn process_checkpoint(&self, checkpoint: CheckpointData) -> Result<()>;
}
```

在此示例中， `CheckpointData` 结构表示完整的检查点内容。该结构包含检查点摘要和内容，以及每个单独事务的详细信息。各个交易数据包括事件和输入/输出对象。此内容的完整定义位于 `sui-types` 箱的 `full_checkpoint_content.rs` 文件中。

## 流式数据源检查点​

索引器的数据摄取支持多个流式数据源检查点​。

### 远程 Reader​

最直接的流式数据源是订阅检查点内容的远程存储。 Mysten Labs 提供以下存储桶：

- 测试网： https://checkpoints.testnet.sui.io
- 主网： https://checkpoints.mainnet.sui.io

## 本地 Reader

将数据摄取守护进程与完整节点并置，并在后者上启用检查点转储以设置本地流式数据源。启用后，完整节点开始将执行的检查点作为文件转储到本地目录，并且数据摄取守护进程通过类似 inotify 的机制订阅目录中的更改。这种方法可以最大限度地减少摄取延迟（在完整节点上的检查点执行器之后立即处理检查点）并摆脱对外部管理存储桶的依赖。

要启用，请将以下内容添加到完整节点配置文件中：

```plain
checkpoint-executor-config:
  checkpoint-execution-max-concurrency: 200
  local-execution-timeout-sec: 30
  data-ingestion-dir: <path to a local directory>
```

### 混合模式​

指定本地和远程存储作为后备，以确保持续的数据流。该框架始终优先考虑本地可用的检查点数据而不是远程数据。当您想要开始利用自己的完整节点进行数据摄取但需要部分回填历史数据或只是进行故障转移时，它非常有用。

## 示例

Sui 数据摄取框架提供了帮助函数来快速引导索引器工作流程。

```rust
struct CustomWorker;

#[async_trait]
impl Worker for CustomWorker {
    async fn process_checkpoint(&self, checkpoint: CheckpointData) -> Result<()> {
        // custom processing logic
        ...
        Ok(())
    }
}

#[tokio::main]
async fn main() -> Result<()> {
    let (executor, term_sender) = setup_single_workflow(
        CustomWorker,
        "https://checkpoints.mainnet.sui.io".to_string(),
        0, /* initial checkpoint number */
        5, /* concurrency */
        None, /* extra reader options */
    ).await?;
    executor.await?;
    Ok(())
}
```

这适用于具有单个摄取管道的设置，其中进度跟踪在框架外部进行管理。

对于更复杂的设置，请参阅以下示例：

```rust
struct CustomWorker;

#[async_trait]
impl Worker for CustomWorker {
    async fn process_checkpoint(&self, checkpoint: CheckpointData) -> Result<()> {
        // custom processing logic
        ...
        Ok(())
    }
}

#[tokio::main]
async fn main() -> Result<()> {
  let (exit_sender, exit_receiver) = oneshot::channel();
  let metrics = DataIngestionMetrics::new(&Registry::new());
  let progress_store = FileProgressStore::new("path_to_file");
  let mut executor = IndexerExecutor::new(progress_store, 1 /* number of workflow types */, metrics);
  let worker_pool = WorkerPool::new(CustomWorker, "custom worker", 100);
  executor.register(worker_pool).await?;
  executor.run(
      PathBuf::from("..."), // path to a local directory
      Some("https://checkpoints.mainnet.sui.io".to_string()),
      vec![], // optional remote store access options
      exit_receiver,
  ).await?;
  Ok(())
}
```

让我们突出显示几行代码：

```rust
let worker_pool = WorkerPool::new(CustomWorker, "custom worker", 100);
executor.register(worker_pool).await?;
```

数据摄取执行器可以同时运行多个工作流程。对于每个工作流程，您需要创建一个单独的工作池并将其注册到执行器中。 WorkerPool 需要 Worker 特征的实例、工作流名称（用于跟踪进度存储和指标中的流程进度）以及并发性。

并发参数指定工作流使用的线程数。当任务是幂等的并且可以并行且无序地处理时，并发值大于 1 会很有帮助。执行器仅在处理完所有先前的检查点后才将进度/水印更新到某个检查点。
