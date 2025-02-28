# 常见问题解答

## 如何升级 Ollama？

macOS 和 Windows 上的 Ollama 会自动下载更新。点击任务栏或菜单栏图标，然后点击"重启以更新"来应用更新。也可以通过[手动](https://ollama.com/download/)下载最新版本来安装更新。

在 Linux 上，重新运行安装脚本：

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

## 如何查看日志？

请查阅[故障排除](./troubleshooting.md)文档了解更多关于使用日志的信息。

## 我的 GPU 是否与 Ollama 兼容？

请参考 [GPU 文档](./gpu.md)。

## 如何指定上下文窗口大小？

默认情况下，Ollama 使用 2048 个 token 的上下文窗口大小。

要在使用 `ollama run` 时更改此设置，请使用 `/set parameter`：

```shell
/set parameter num_ctx 4096
```

在使用 API 时，指定 `num_ctx` 参数：

```shell
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Why is the sky blue?",
  "options": {
    "num_ctx": 4096
  }
}'
```

## 如何确认我的模型已加载到 GPU 上？

使用 `ollama ps` 命令查看当前加载到内存中的模型。

```shell
ollama ps
```

> **输出**:
>
> ```
> NAME      	ID          	SIZE 	PROCESSOR	UNTIL
> llama3:70b	bcfb190ca3a7	42 GB	100% GPU 	4 minutes from now
> ```

`Processor` 列会显示模型加载的位置：
* `100% GPU` 表示模型完全加载到了 GPU 中
* `100% CPU` 表示模型完全加载到了系统内存中
* `48%/52% CPU/GPU` 表示模型部分加载到了 GPU 和系统内存中

## 如何配置 Ollama 服务器？

Ollama 服务器可以通过环境变量进行配置。

### 在 Mac 上设置环境变量

如果 Ollama 作为 macOS 应用程序运行，环境变量应使用 `launchctl` 设置：

1. 对每个环境变量，调用 `launchctl setenv`。

    ```bash
    launchctl setenv OLLAMA_HOST "0.0.0.0:11434"
    ```

2. 重启 Ollama 应用程序。

### 在 Linux 上设置环境变量

如果 Ollama 作为 systemd 服务运行，环境变量应使用 `systemctl` 设置：

1. 通过调用 `systemctl edit ollama.service` 编辑 systemd 服务。这将打开一个编辑器。

2. 在 `[Service]` 部分下，为每个环境变量添加一行 `Environment`：

    ```ini
    [Service]
    Environment="OLLAMA_HOST=0.0.0.0:11434"
    ```

3. 保存并退出。

4. 重新加载 `systemd` 并重启 Ollama：

   ```shell
   systemctl daemon-reload
   systemctl restart ollama
   ```

### 在 Windows 上设置环境变量

在 Windows 上，Ollama 继承您的用户和系统环境变量。

1. 首先通过点击任务栏中的图标退出 Ollama。

2. 启动设置（Windows 11）或控制面板（Windows 10）应用程序并搜索"环境变量"。

3. 点击"编辑您账户的环境变量"。

4. 为您的用户账户编辑或创建 `OLLAMA_HOST`、`OLLAMA_MODELS` 等新变量。

5. 点击确定/应用以保存。

6. 从 Windows 开始菜单启动 Ollama 应用程序。

## 如何在代理后面使用 Ollama？

Ollama 从互联网拉取模型，可能需要代理服务器来访问这些模型。使用 `HTTPS_PROXY` 通过代理重定向出站请求。确保将代理证书安装为系统证书。有关如何在您的平台上使用环境变量，请参考上面的部分。

> [!注意]
> 避免设置 `HTTP_PROXY`。Ollama 不使用 HTTP 拉取模型，只使用 HTTPS。设置 `HTTP_PROXY` 可能会中断客户端与服务器的连接。

### 如何在 Docker 中通过代理使用 Ollama？

Ollama Docker 容器镜像可以通过在启动容器时传递 `-e HTTPS_PROXY=https://proxy.example.com` 来配置使用代理。

或者，Docker 守护程序可以配置为使用代理。Docker Desktop 在 [macOS](https://docs.docker.com/desktop/settings/mac/#proxies)、[Windows](https://docs.docker.com/desktop/settings/windows/#proxies) 和 [Linux](https://docs.docker.com/desktop/settings/linux/#proxies) 上的说明，以及使用 [systemd 的 Docker 守护程序](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)的说明都已提供。

使用 HTTPS 时，确保证书安装为系统证书。使用自签名证书时可能需要新的 Docker 镜像。

```dockerfile
FROM ollama/ollama
COPY my-ca.pem /usr/local/share/ca-certificates/my-ca.crt
RUN update-ca-certificates
```

构建并运行此镜像：

```shell
docker build -t ollama-with-ca .
docker run -d -e HTTPS_PROXY=https://my.proxy.example.com -p 11434:11434 ollama-with-ca
```

## Ollama 会将我的提示和回答发送回 ollama.com 吗？

不会。Ollama 在本地运行，对话数据不会离开您的机器。

## 如何在我的网络上暴露 Ollama？

Ollama 默认绑定到 127.0.0.1 端口 11434。使用 `OLLAMA_HOST` 环境变量更改绑定地址。

有关如何在您的平台上设置环境变量，请参考[上面](#how-do-i-configure-ollama-server)的部分。

## 如何使用代理服务器与 Ollama？

Ollama 运行一个 HTTP 服务器，可以使用 Nginx 等代理服务器暴露。为此，配置代理来转发请求并可选择设置所需的头信息（如果不在网络上暴露 Ollama）。例如，使用 Nginx：

```nginx
server {
    listen 80;
    server_name example.com;  # 替换为您的域名或 IP
    location / {
        proxy_pass http://localhost:11434;
        proxy_set_header Host localhost:11434;
    }
}
```

## 如何使用 ngrok 与 Ollama？

Ollama 可以使用多种隧道工具访问。例如，使用 Ngrok：

```shell
ngrok http 11434 --host-header="localhost:11434"
```

## 如何使用 Cloudflare Tunnel 与 Ollama？

要使用 Cloudflare Tunnel 与 Ollama，请使用 `--url` 和 `--http-host-header` 标志：

```shell
cloudflared tunnel --url http://localhost:11434 --http-host-header="localhost:11434"
```

## 如何允许其他网络源访问 Ollama？

Ollama 默认允许来自 `127.0.0.1` 和 `0.0.0.0` 的跨源请求。可以使用 `OLLAMA_ORIGINS` 配置其他源。

有关如何在您的平台上设置环境变量，请参考[上面](#how-do-i-configure-ollama-server)的部分。

## 模型存储在哪里？

- macOS: `~/.ollama/models`
- Linux: `/usr/share/ollama/.ollama/models`
- Windows: `C:\Users\%username%\.ollama\models`

### 如何将它们设置到不同位置？

如果需要使用不同的目录，请将环境变量 `OLLAMA_MODELS` 设置为所选目录。

> 注意：在 Linux 上使用标准安装程序时，`ollama` 用户需要对指定目录有读写权限。要将目录分配给 `ollama` 用户，请运行 `sudo chown -R ollama:ollama <directory>`。

有关如何在您的平台上设置环境变量，请参考[上面](#how-do-i-configure-ollama-server)的部分。

## 如何在 Visual Studio Code 中使用 Ollama？

已经有大量针对 VSCode 以及其他编辑器的插件可用，它们利用 Ollama。请查看主仓库 readme 底部的[扩展和插件](https://github.com/ollama/ollama#extensions--plugins)列表。

## 如何在 Docker 中使用 GPU 加速的 Ollama？

Ollama Docker 容器可以在 Linux 或 Windows（使用 WSL2）中配置 GPU 加速。这需要 [nvidia-container-toolkit](https://github.com/NVIDIA/nvidia-container-toolkit)。有关更多详情，请参阅 [ollama/ollama](https://hub.docker.com/r/ollama/ollama)。

由于缺乏 GPU 透传和模拟，macOS 的 Docker Desktop 不支持 GPU 加速。

## 为什么在 Windows 10 上 WSL2 的网络很慢？

这可能会影响安装 Ollama 以及下载模型。

打开"控制面板 > 网络和互联网 > 查看网络状态和任务"，点击左侧面板上的"更改适配器设置"。找到"vEthernel (WSL)"适配器，右键点击并选择"属性"。
点击"配置"并打开"高级"选项卡。在属性中搜索"Large Send Offload Version 2 (IPv4)"和"Large Send Offload Version 2 (IPv6)"。禁用这两个属性。

## 如何预加载模型到 Ollama 中以获得更快的响应时间？

如果您使用 API，可以通过向 Ollama 服务器发送空请求来预加载模型。这适用于 `/api/generate` 和 `/api/chat` API 端点。

要使用生成端点预加载 mistral 模型，请使用：

```shell
curl http://localhost:11434/api/generate -d '{"model": "mistral"}'
```

要使用聊天完成端点，请使用：

```shell
curl http://localhost:11434/api/chat -d '{"model": "mistral"}'
```

要使用 CLI 预加载模型，请使用以下命令：

```shell
ollama run llama3.2 ""
```

## 如何保持模型加载在内存中或使其立即卸载？

默认情况下，模型在内存中保留 5 分钟后才会卸载。如果您向 LLM 发出多个请求，这可以提供更快的响应时间。如果您想立即从内存中卸载模型，请使用 `ollama stop` 命令：

```shell
ollama stop llama3.2
```

如果您使用 API，可以使用 `/api/generate` 和 `/api/chat` 端点的 `keep_alive` 参数来设置模型在内存中保留的时间。`keep_alive` 参数可以设置为：
* 一个持续时间字符串（如 "10m" 或 "24h"）
* 以秒为单位的数字（如 3600）
* 任何负数，这将使模型保持加载在内存中（例如 -1 或 "-1m"）
* '0'，这将在生成响应后立即卸载模型

例如，要预加载模型并将其保留在内存中，请使用：

```shell
curl http://localhost:11434/api/generate -d '{"model": "llama3.2", "keep_alive": -1}'
```

要卸载模型并释放内存，请使用：

```shell
curl http://localhost:11434/api/generate -d '{"model": "llama3.2", "keep_alive": 0}'
```

或者，您可以通过在启动 Ollama 服务器时设置 `OLLAMA_KEEP_ALIVE` 环境变量来更改所有模型加载到内存中的时间。`OLLAMA_KEEP_ALIVE` 变量使用与上述 `keep_alive` 参数相同的参数类型。有关如何正确设置环境变量，请参考[如何配置 Ollama 服务器](#how-do-i-configure-ollama-server)部分的说明。

使用 `/api/generate` 和 `/api/chat` API 端点的 `keep_alive` API 参数将覆盖 `OLLAMA_KEEP_ALIVE` 设置。

## 如何管理 Ollama 服务器可以排队的最大请求数？

如果向服务器发送了太多请求，它将返回 503 错误，表明服务器过载。您可以通过设置 `OLLAMA_MAX_QUEUE` 来调整可以排队的请求数量。

## Ollama 如何处理并发请求？

Ollama 支持两个级别的并发处理。如果您的系统有足够的可用内存（使用 CPU 推理时的系统内存，或用于 GPU 推理的 VRAM），则可以同时加载多个模型。对于给定的模型，如果在加载模型时有足够的可用内存，则可以配置为允许并行请求处理。

如果在一个或多个模型已经加载的情况下，没有足够的可用内存来加载新的模型请求，所有新请求将排队等待，直到可以加载新模型。随着先前模型变为空闲，将卸载一个或多个模型以为新模型腾出空间。排队的请求将按顺序处理。使用 GPU 推理时，新模型必须能够完全放入 VRAM 中才能允许并发模型加载。

给定模型的并行请求处理会导致上下文大小增加到并行请求的数量。例如，2K 上下文与 4 个并行请求将导致 8K 上下文和额外的内存分配。

以下服务器设置可用于调整 Ollama 如何在大多数平台上处理并发请求：

- `OLLAMA_MAX_LOADED_MODELS` - 可以同时加载的模型的最大数量，前提是它们适合可用内存。默认值为 3 * GPU 数量，或对于 CPU 推理为 3。
- `OLLAMA_NUM_PARALLEL` - 每个模型将同时处理的最大并行请求数。默认值将根据可用内存自动选择 4 或 1。
- `OLLAMA_MAX_QUEUE` - Ollama 在繁忙时拒绝额外请求之前将排队的最大请求数。默认值为 512。

注意：由于 ROCm v5.7 中对可用 VRAM 报告的限制，带有 Radeon GPU 的 Windows 当前默认最多 1 个模型。一旦 ROCm v6.2 可用，Windows Radeon 将遵循上述默认设置。您可以在 Windows 的 Radeon 上启用并发模型加载，但请确保不要加载超过 GPU VRAM 容量的模型。

## Ollama 如何在多个 GPU 上加载模型？

在加载新模型时，Ollama 会评估模型所需的 VRAM 与当前可用的 VRAM。如果模型可以完全放入任何单个 GPU，Ollama 将在该 GPU 上加载模型。这通常提供最佳性能，因为它减少了推理过程中通过 PCI 总线传输数据的量。如果模型不能完全放入一个 GPU，则它将分散到所有可用的 GPU 上。

## 如何启用闪存注意力（Flash Attention）？

闪存注意力是大多数现代模型的一个特性，可以随着上下文大小的增长显著减少内存使用。要启用闪存注意力，在启动 Ollama 服务器时将 `OLLAMA_FLASH_ATTENTION` 环境变量设置为 `1`。

## 如何为 K/V 缓存设置量化类型？

K/V 上下文缓存可以量化以在启用闪存注意力时显著减少内存使用。

要使用量化的 K/V 缓存与 Ollama，您可以设置以下环境变量：

- `OLLAMA_KV_CACHE_TYPE` - K/V 缓存的量化类型。默认为 `f16`。

> 注意：目前这是一个全局选项 - 意味着所有模型都将使用指定的量化类型运行。

当前可用的 K/V 缓存量化类型有：

- `f16` - 高精度和内存使用（默认）。
- `q8_0` - 8 位量化，使用约 `f16` 一半的内存，精度损失很小，这通常对模型质量没有明显影响（如果不使用 f16，推荐使用）。
- `q4_0` - 4 位量化，使用约 `f16` 四分之一的内存，精度损失中等，在更高的上下文大小时可能更明显。

缓存量化对模型响应质量的影响程度将取决于模型和任务。具有高 GQA 计数的模型（例如 Qwen2）可能会比具有低 GQA 计数的模型受到量化导致的精度影响更大。

您可能需要尝试不同的量化类型，以找到内存使用与质量之间的最佳平衡。
