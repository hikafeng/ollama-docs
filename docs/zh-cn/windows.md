# Ollama Windows

欢迎使用 Windows 版 Ollama。

不再需要 WSL！

Ollama 现在作为原生 Windows 应用运行，包括对 NVIDIA 和 AMD Radeon GPU 的支持。
安装 Windows 版 Ollama 后，Ollama 将在后台运行，
`ollama` 命令行可在 `cmd`、`powershell` 或您喜欢的
终端应用中使用。和往常一样，Ollama [api](./api.md) 将在
`http://localhost:11434` 上提供服务。

## 系统要求

* Windows 10 22H2 或更新版本，家庭版或专业版
* 如果您有 NVIDIA 显卡，需要 NVIDIA 452.39 或更新的驱动程序
* 如果您有 Radeon 显卡，需要 AMD Radeon 驱动程序 <https://www.amd.com/en/support>

Ollama 使用 Unicode 字符来指示进度，在 Windows 10 的一些较旧的终端字体中可能显示为未知方块。如果您看到这种情况，请尝试更改终端字体设置。

## 获取安装包

打开 [Windows 安装包](https://ollama.com/download/windows) 下载

## 文件系统要求

Ollama 安装不需要管理员权限，默认安装在您的主目录中。您至少需要 4GB 的空间用于二进制安装。安装 Ollama 后，您还需要额外的空间来存储大型语言模型，这些模型可能占用几十到几百 GB 的空间。如果您的主目录没有足够的空间，您可以更改二进制文件的安装位置和模型的存储位置。

### 更改安装位置

要将 Ollama 应用程序安装在不同于主目录的位置，请使用以下标志启动安装程序

```powershell
OllamaSetup.exe /DIR="d:\some\location"
```

### 更改模型位置

要更改 Ollama 存储下载模型的位置而不使用主目录，请在用户账户中设置环境变量 `OLLAMA_MODELS`。

1. 启动设置（Windows 11）或控制面板（Windows 10）应用程序，并搜索_环境变量_。

2. 点击_编辑您账户的环境变量_。

3. 为您的用户账户编辑或创建一个名为 `OLLAMA_MODELS` 的新变量，指向您想要存储模型的位置。

4. 点击确定/应用保存。

如果 Ollama 已经在运行，请退出托盘应用程序并从开始菜单重新启动它，或者在保存环境变量后启动一个新的终端。

## API 访问

以下是一个从 `powershell` 访问 API 的快速示例

```powershell
(Invoke-WebRequest -method POST -Body '{"model":"llama3.2", "prompt":"Why is the sky blue?", "stream": false}' -uri http://localhost:11434/api/generate ).Content | ConvertFrom-json
```

## 故障排除

Windows 版 Ollama 将文件存储在几个不同的位置。您可以通过按下 `<Ctrl>+R` 并输入以下内容在资源管理器窗口中查看它们：

* `explorer %LOCALAPPDATA%\Ollama` 包含日志和下载的更新
  * *app.log* 包含 GUI 应用程序的最新日志
  * *server.log* 包含最新的服务器日志
  * *upgrade.log* 包含升级的日志输出
* `explorer %LOCALAPPDATA%\Programs\Ollama` 包含二进制文件（安装程序将其添加到您的用户 PATH 中）
* `explorer %HOMEPATH%\.ollama` 包含模型和配置
* `explorer %TEMP%` 包含临时可执行文件，位于一个或多个 `ollama*` 目录中

## 卸载

Ollama Windows 安装程序注册了一个卸载应用程序。在 Windows 设置的"添加或删除程序"中，您可以卸载 Ollama。

> [!NOTE]
> 如果您已[更改了 OLLAMA_MODELS 位置](#更改模型位置)，安装程序不会删除您下载的模型。

## 独立 CLI

在 Windows 上安装 Ollama 最简单的方法是使用 `OllamaSetup.exe` 安装程序。它可以在您的账户中安装，无需管理员权限。我们定期更新 Ollama 以支持最新的模型，这个安装程序将帮助您保持更新。

如果您想将 Ollama 作为服务安装或集成，可以使用 `ollama-windows-amd64.zip` 压缩文件，它仅包含 Ollama CLI 和 Nvidia 与 AMD 的 GPU 库依赖项。这允许将 Ollama 嵌入到现有应用程序中，或通过工具如 [NSSM](https://nssm.cc/) 使用 `ollama serve` 作为系统服务运行它。

> [!NOTE]
> 如果您正在从之前的版本升级，应该先删除旧目录。
