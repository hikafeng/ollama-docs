# Ollama Docker 镜像

### 仅 CPU 版本

```shell
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

### Nvidia GPU 版本
安装 [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#installation)。

#### 通过 Apt 安装
1. 配置仓库

    ```shell
    curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
        | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
    curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
        | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
        | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
    sudo apt-get update
    ```

2. 安装 NVIDIA Container Toolkit 包

    ```shell
    sudo apt-get install -y nvidia-container-toolkit
    ```

#### 通过 Yum 或 Dnf 安装
1. 配置仓库

    ```shell
    curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo \
        | sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo
    ```

2. 安装 NVIDIA Container Toolkit 包

    ```shell
    sudo yum install -y nvidia-container-toolkit
    ```

#### 配置 Docker 使用 Nvidia 驱动

```shell
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

#### 启动容器

```shell
docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

> [!注意]  
> 如果您在 NVIDIA JetPack 系统上运行，Ollama 无法自动识别正确的 JetPack 版本。请向容器传递环境变量 JETSON_JETPACK=5 或 JETSON_JETPACK=6 以选择版本 5 或 6。

### AMD GPU 版本

要使用 Docker 和 AMD GPU 运行 Ollama，请使用 `rocm` 标签和以下命令：

```shell
docker run -d --device /dev/kfd --device /dev/dri -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama:rocm
```

### 本地运行模型

现在您可以运行模型：

```shell
docker exec -it ollama ollama run llama3.2
```

### 尝试不同的模型

更多模型可以在 [Ollama 模型库](https://ollama.com/library) 中找到。
