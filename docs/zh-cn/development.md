# 开发

安装前置条件：

- [Go](https://go.dev/doc/install)
- C/C++ 编译器，例如 macOS 上的 Clang，[TDM-GCC](https://github.com/jmeubank/tdm-gcc/releases/latest)（Windows amd64）或 [llvm-mingw](https://github.com/mstorsjo/llvm-mingw)（Windows arm64），Linux 上的 GCC/Clang。

然后从仓库的根目录构建并运行 Ollama：

```shell
go run . serve
```

## macOS（Apple 芯片）

macOS Apple 芯片支持 Metal，这已内置于 Ollama 二进制文件中。无需额外步骤。

## macOS（Intel）

安装前置条件：

- [CMake](https://cmake.org/download/) 或 `brew install cmake`

然后，配置并构建项目：

```shell
cmake -B build
cmake --build build
```

最后，运行 Ollama：

```shell
go run . serve
```

## Windows

安装前置条件：

- [CMake](https://cmake.org/download/)
- [Visual Studio 2022](https://visualstudio.microsoft.com/downloads/) 包括本机桌面工作负载
- （可选）AMD GPU 支持
    - [ROCm](https://rocm.docs.amd.com/en/latest/)
    - [Ninja](https://github.com/ninja-build/ninja/releases)
- （可选）NVIDIA GPU 支持
    - [CUDA SDK](https://developer.nvidia.com/cuda-downloads?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_network)

然后，配置并构建项目：

```shell
cmake -B build
cmake --build build --config Release
```

> [!IMPORTANT]
> 为 ROCm 构建需要额外标志：
> ```
> cmake -B build -G Ninja -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
> cmake --build build --config Release
> ```


最后，运行 Ollama：

```shell
go run . serve
```

## Windows（ARM）

Windows ARM 目前不支持额外的加速库。不要使用 cmake，只需 `go run` 或 `go build`。

## Linux

安装前置条件：

- [CMake](https://cmake.org/download/) 或 `sudo apt install cmake` 或 `sudo dnf install cmake`
- （可选）AMD GPU 支持
    - [ROCm](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/quick-start.html)
- （可选）NVIDIA GPU 支持
    - [CUDA SDK](https://developer.nvidia.com/cuda-downloads)

> [!IMPORTANT]
> 运行 CMake 前确保前置条件在 `PATH` 中。


然后，配置并构建项目：

```shell
cmake -B build
cmake --build build
```

最后，运行 Ollama：

```shell
go run . serve
```

## Docker

```shell
docker build .
```

### ROCm

```shell
docker build --build-arg FLAVOR=rocm .
```

## 运行测试

要运行测试，使用 `go test`：

```shell
go test ./...
```

## 库检测

Ollama 在相对于 `ollama` 可执行文件的以下路径中查找加速库：

* `./lib/ollama`（Windows）
* `../lib/ollama`（Linux）
* `.`（macOS）
* `build/lib/ollama`（用于开发）

如果找不到这些库，Ollama 将不会使用任何加速库运行。
