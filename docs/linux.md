# Linux

## 安装

要安装 Ollama，请运行以下命令：

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

## 手动安装

> [!注意]
> 如果您正在从先前版本升级，应该先使用 `sudo rm -rf /usr/lib/ollama` 删除旧库文件。

下载并解压安装包：

```shell
curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
sudo tar -C /usr -xzf ollama-linux-amd64.tgz
```

启动 Ollama：

```shell
ollama serve
```

在另一个终端中，验证 Ollama 是否运行：

```shell
ollama -v
```

### AMD GPU 安装

如果您有 AMD GPU，还需要下载并解压额外的 ROCm 包：

```shell
curl -L https://ollama.com/download/ollama-linux-amd64-rocm.tgz -o ollama-linux-amd64-rocm.tgz
sudo tar -C /usr -xzf ollama-linux-amd64-rocm.tgz
```

### ARM64 安装

下载并解压 ARM64 专用安装包：

```shell
curl -L https://ollama.com/download/ollama-linux-arm64.tgz -o ollama-linux-arm64.tgz
sudo tar -C /usr -xzf ollama-linux-arm64.tgz
```

### 将 Ollama 添加为开机启动服务（推荐）

为 Ollama 创建用户和用户组：

```shell
sudo useradd -r -s /bin/false -U -m -d /usr/share/ollama ollama
sudo usermod -a -G ollama $(whoami)
```

在 `/etc/systemd/system/ollama.service` 创建服务文件：

```ini
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=$PATH"

[Install]
WantedBy=default.target
```

然后启动服务：

```shell
sudo systemctl daemon-reload
sudo systemctl enable ollama
```

### 安装 CUDA 驱动（可选）

[下载并安装](https://developer.nvidia.com/cuda-downloads) CUDA。

运行以下命令验证驱动是否已安装，此命令应显示您 GPU 的详细信息：

```shell
nvidia-smi
```

### 安装 AMD ROCm 驱动（可选）

[下载并安装](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/tutorial/quick-start.html) ROCm v6。

### 启动 Ollama

启动 Ollama 并验证其运行状态：

```shell
sudo systemctl start ollama
sudo systemctl status ollama
```

> [!注意]
> 虽然 AMD 已经将 `amdgpu` 驱动贡献给了官方 Linux 内核源代码，但该版本较旧，
> 可能不支持所有 ROCm 功能。我们建议您从 <https://www.amd.com/en/support/linux-drivers>
> 安装最新的驱动程序，以获得对 Radeon GPU 的最佳支持。

## 自定义

要自定义 Ollama 的安装，您可以通过运行以下命令编辑 systemd 服务文件或环境变量：

```shell
sudo systemctl edit ollama
```

或者在 `/etc/systemd/system/ollama.service.d/override.conf` 手动创建覆盖文件：

```ini
[Service]
Environment="OLLAMA_DEBUG=1"
```

## 更新

通过再次运行安装脚本更新 Ollama：

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

或者重新下载 Ollama：

```shell
curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
sudo tar -C /usr -xzf ollama-linux-amd64.tgz
```

## 安装特定版本

使用 `OLLAMA_VERSION` 环境变量与安装脚本一起安装特定版本的 Ollama，包括预发布版本。您可以在[发布页面](https://github.com/ollama/ollama/releases)中找到版本号。

例如：

```shell
curl -fsSL https://ollama.com/install.sh | OLLAMA_VERSION=0.5.7 sh
```

## 查看日志

要查看作为启动服务运行的 Ollama 的日志，请运行：

```shell
journalctl -e -u ollama
```

## 卸载

移除 Ollama 服务：

```shell
sudo systemctl stop ollama
sudo systemctl disable ollama
sudo rm /etc/systemd/system/ollama.service
```

从您的 bin 目录中删除 Ollama 二进制文件（可能在 `/usr/local/bin`、`/usr/bin` 或 `/bin` 中）：

```shell
sudo rm $(which ollama)
```

删除下载的模型以及 Ollama 服务用户和用户组：

```shell
sudo rm -r /usr/share/ollama
sudo userdel ollama
sudo groupdel ollama
```

删除已安装的库：

```shell
sudo rm -rf /usr/local/lib/ollama
```
