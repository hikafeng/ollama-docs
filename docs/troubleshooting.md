# 如何排除故障

有时 Ollama 可能不会按预期工作。找出问题原因的最好方法之一是查看日志。在 **Mac** 上，可以通过运行以下命令查看日志：

```shell
cat ~/.ollama/logs/server.log
```

在使用 systemd 的 **Linux** 系统上，可以通过以下命令查看日志：

```shell
journalctl -u ollama --no-pager
```

当您在**容器**中运行 Ollama 时，日志会输出到容器的 stdout/stderr：

```shell
docker logs <容器名称>
```

（使用 `docker ps` 查找容器名称）

如果在终端中手动运行 `ollama serve`，日志将显示在该终端中。

当您在 **Windows** 上运行 Ollama 时，日志存储在几个不同的位置。您可以通过按下 `<cmd>+R` 并输入以下路径在资源管理器中查看它们：
- `explorer %LOCALAPPDATA%\Ollama` 查看日志。最新的服务器日志将在 `server.log` 中，较旧的日志将在 `server-#.log` 中
- `explorer %LOCALAPPDATA%\Programs\Ollama` 浏览二进制文件（安装程序会将其添加到用户 PATH 中）
- `explorer %HOMEPATH%\.ollama` 浏览模型和配置存储位置
- `explorer %TEMP%` 查看临时可执行文件存储在一个或多个 `ollama*` 目录中

要启用额外的调试日志以帮助排除问题，首先**从托盘菜单退出运行应用程序**，然后在 PowerShell 终端中运行：

```powershell
$env:OLLAMA_DEBUG="1"
& "ollama app.exe"
```

加入 [Discord](https://discord.gg/ollama) 获取日志解读帮助。

## LLM 库

Ollama 包含针对不同 GPU 和 CPU 向量特性编译的多个 LLM 库。Ollama 会根据系统功能尝试选择最佳的库。如果自动检测出现问题，或者遇到其他问题（例如 GPU 崩溃），您可以通过强制使用特定的 LLM 库来解决。`cpu_avx2` 性能最佳，其次是 `cpu_avx`，而最慢但兼容性最好的是 `cpu`。MacOS 下的 Rosetta 仿真可以使用 `cpu` 库。

在服务器日志中，您将看到类似这样的消息（因发行版而异）：

```
Dynamic LLM libraries [rocm_v6 cpu cpu_avx cpu_avx2 cuda_v11 rocm_v5]
```

**实验性 LLM 库覆盖**

您可以将 OLLAMA_LLM_LIBRARY 设置为任何可用的 LLM 库，以绕过自动检测。例如，如果您有 CUDA 卡，但希望强制使用支持 AVX2 向量的 CPU LLM 库，可以使用：

```shell
OLLAMA_LLM_LIBRARY="cpu_avx2" ollama serve
```

您可以通过以下命令查看 CPU 具有哪些功能：

```shell
cat /proc/cpuinfo| grep flags | head -1
```

## 在 Linux 上安装旧版本或预发布版本

如果您在 Linux 上遇到问题并想要安装旧版本，或者想在正式发布前尝试预发布版本，可以告诉安装脚本要安装的版本：

```shell
curl -fsSL https://ollama.com/install.sh | OLLAMA_VERSION=0.5.7 sh
```

## Linux tmp noexec 问题

如果您的系统在 Ollama 存储临时可执行文件的位置配置了 "noexec" 标志，您可以通过设置 OLLAMA_TMPDIR 指定 Ollama 运行用户可写的备用位置。例如 OLLAMA_TMPDIR=/usr/share/ollama/

## Linux docker 问题

如果 Ollama 最初在 docker 容器中的 GPU 上运行正常，但在一段时间后切换到 CPU 运行，服务器日志中报告 GPU 发现失败错误，这可以通过在 Docker 中禁用 systemd cgroup 管理来解决。在主机上编辑 `/etc/docker/daemon.json` 并将 `"exec-opts": ["native.cgroupdriver=cgroupfs"]` 添加到 docker 配置中。

## NVIDIA GPU 识别

当 Ollama 启动时，它会对系统中存在的 GPU 进行清点，以确定兼容性和可用的 VRAM 容量。有时这种识别可能找不到您的 GPU。通常，运行最新驱动程序将获得最佳结果。

### Linux NVIDIA 故障排除

如果您使用容器运行 Ollama，请确保先按照 [docker.md](./docker.md) 中描述的方式设置容器运行时。

有时 Ollama 可能难以初始化 GPU。当您检查服务器日志时，这可能表现为各种错误代码，如 "3"（未初始化）、"46"（设备不可用）、"100"（无设备）、"999"（未知）或其他。以下故障排除技术可能有助于解决问题：

- 如果您使用的是容器，容器运行时是否正常工作？尝试 `docker run --gpus all ubuntu nvidia-smi` - 如果这不起作用，Ollama 将无法看到您的 NVIDIA GPU。
- uvm 驱动程序是否已加载？`sudo nvidia-modprobe -u`
- 尝试重新加载 nvidia_uvm 驱动程序 - `sudo rmmod nvidia_uvm` 然后 `sudo modprobe nvidia_uvm`
- 尝试重新启动
- 确保您运行的是最新的 nvidia 驱动程序

如果这些方法都无法解决问题，请收集额外信息并提交问题：
- 设置 `CUDA_ERROR_LEVEL=50` 并重试以获取更多诊断日志
- 检查 dmesg 是否有任何错误 `sudo dmesg | grep -i nvrm` 和 `sudo dmesg | grep -i nvidia`

## AMD GPU 识别

在 Linux 上，AMD GPU 访问通常需要 `video` 和/或 `render` 组成员资格才能访问 `/dev/kfd` 设备。如果权限设置不正确，Ollama 将检测到这一点并在服务器日志中报告错误。

在容器中运行时，在某些 Linux 发行版和容器运行时中，ollama 进程可能无法访问 GPU。使用 `ls -lnd /dev/kfd /dev/dri /dev/dri/*` 在主机系统上确定系统上的**数字**组 ID，并向容器传递额外的 `--group-add ...` 参数，以便它可以访问所需设备。例如，在以下输出中 `crw-rw---- 1 0 44 226, 0 Sep 16 16:55 /dev/dri/card0` 组 ID 列是 `44`。

如果您在让 Ollama 正确识别或使用 GPU 进行推理时遇到问题，以下方法可能有助于查明故障：
- `AMD_LOG_LEVEL=3` 在 AMD HIP/ROCm 库中启用信息日志级别。这可以帮助显示更详细的错误代码，有助于排除故障
- `OLLAMA_DEBUG=1` 在 GPU 识别过程中将报告额外信息
- 检查 dmesg 中是否有来自 amdgpu 或 kfd 驱动程序的任何错误 `sudo dmesg | grep -i amdgpu` 和 `sudo dmesg | grep -i kfd`

## 多个 AMD GPU

如果在 Linux 上跨多个 AMD GPU 加载模型时遇到乱码响应，请参阅以下指南：

- https://rocm.docs.amd.com/projects/radeon/en/latest/docs/install/native_linux/mgpu.html#mgpu-known-issues-and-limitations

## Windows 终端错误

已知较旧版本的 Windows 10（例如 21H1）在标准终端程序中存在一个不能正确显示控制字符的错误。这可能导致显示一长串类似 `←[?25h←[?25l` 的字符串，有时会出现 `参数不正确` 的错误。要解决此问题，请更新到 Win 10 22H1 或更新版本。
