# 导入模型

## 目录

* [导入 Safetensors 适配器](#导入-Safetensors-微调适配器)
* [导入 Safetensors 模型](#从-Safetensors-权重导入模型)
* [导入 GGUF 文件](#导入基于-GGUF-的模型或适配器)
* [在 ollama.com 上共享模型](#在-ollamacom-上共享您的模型)

## 导入 Safetensors 微调适配器

首先，创建一个包含指向您用于微调的基础模型的 `FROM` 命令的 `Modelfile`，以及指向 Safetensors 适配器目录的 `ADAPTER` 命令：

```dockerfile
FROM <基础模型名称>
ADAPTER /path/to/safetensors/adapter/directory
```

确保在 `FROM` 命令中使用与创建适配器时相同的基础模型，否则会得到不稳定的结果。大多数框架使用不同的量化方法，因此最好使用非量化（即非 QLoRA）适配器。如果您的适配器与 `Modelfile` 在同一目录中，使用 `ADAPTER .` 来指定适配器路径。

现在从创建 `Modelfile` 的目录运行 `ollama create`：

```shell
ollama create my-model
```

最后，测试模型：

```shell
ollama run my-model
```

Ollama 支持导入基于多种不同模型架构的适配器，包括：

* Llama（包括 Llama 2、Llama 3、Llama 3.1 和 Llama 3.2）；
* Mistral（包括 Mistral 1、Mistral 2 和 Mixtral）；以及
* Gemma（包括 Gemma 1 和 Gemma 2）

您可以使用能够输出 Safetensors 格式适配器的微调框架或工具创建适配器，例如：

* Hugging Face [微调框架](https://huggingface.co/docs/transformers/en/training)
* [Unsloth](https://github.com/unslothai/unsloth)
* [MLX](https://github.com/ml-explore/mlx)

## 从 Safetensors 权重导入模型

首先，创建一个包含指向 Safetensors 权重目录的 `FROM` 命令的 `Modelfile`：

```dockerfile
FROM /path/to/safetensors/directory
```

如果您在权重所在的同一目录中创建 Modelfile，可以使用命令 `FROM .`。

现在从创建 `Modelfile` 的目录运行 `ollama create` 命令：

```shell
ollama create my-model
```

最后，测试模型：

```shell
ollama run my-model
```

Ollama 支持导入多种不同架构的模型，包括：

* Llama（包括 Llama 2、Llama 3、Llama 3.1 和 Llama 3.2）；
* Mistral（包括 Mistral 1、Mistral 2 和 Mixtral）；
* Gemma（包括 Gemma 1 和 Gemma 2）；以及
* Phi3

这包括导入基础模型以及与基础模型融合的任何微调模型。

## 导入基于 GGUF 的模型或适配器

如果您有基于 GGUF 的模型或适配器，可以将其导入到 Ollama 中。您可以通过以下方式获取 GGUF 模型或适配器：

* 使用 Llama.cpp 中的 `convert_hf_to_gguf.py` 转换 Safetensors 模型；
* 使用 Llama.cpp 中的 `convert_lora_to_gguf.py` 转换 Safetensors 适配器；或
* 从 HuggingFace 等平台下载模型或适配器

要导入 GGUF 模型，创建一个包含以下内容的 `Modelfile`：

```dockerfile
FROM /path/to/file.gguf
```

对于 GGUF 适配器，创建包含以下内容的 `Modelfile`：

```dockerfile
FROM <模型名称>
ADAPTER /path/to/file.gguf
```

导入 GGUF 适配器时，重要的是使用与创建适配器时相同的基础模型。您可以使用：

* Ollama 中的模型
* GGUF 文件
* 基于 Safetensors 的模型

创建 `Modelfile` 后，使用 `ollama create` 命令构建模型。

```shell
ollama create my-model
```

## 量化模型

量化模型允许您以降低精度为代价，更快地运行模型并减少内存消耗。这使您能够在更普通的硬件上运行模型。

Ollama 可以使用 `ollama create` 命令的 `-q/--quantize` 标志，将 FP16 和 FP32 基础模型量化为不同的量化级别。

首先，创建一个包含您希望量化的 FP16 或 FP32 基础模型的 Modelfile。

```dockerfile
FROM /path/to/my/gemma/f16/model
```

然后使用 `ollama create` 创建量化模型。

```shell
$ ollama create --quantize q4_K_M mymodel
transferring model data
quantizing F16 model to Q4_K_M
creating new layer sha256:735e246cc1abfd06e9cdcf95504d6789a6cd1ad7577108a70d9902fef503c1bd
creating new layer sha256:0853f0ad24e5865173bbf9ffcc7b0f5d56b66fd690ab1009867e45e7d2c4db0f
writing manifest
success
```

### 支持的量化方法

* `q4_0`
* `q4_1`
* `q5_0`
* `q5_1`
* `q8_0`

#### K-means 量化

* `q3_K_S`
* `q3_K_M`
* `q3_K_L`
* `q4_K_S`
* `q4_K_M`
* `q5_K_S`
* `q5_K_M`
* `q6_K`

## 在 ollama.com 上共享您的模型

您可以通过将创建的任何模型推送到 [ollama.com](https://ollama.com)，使其他用户能够尝试使用。

首先，使用浏览器访问 [Ollama 注册](https://ollama.com/signup) 页面。如果您已有账户，可以跳过此步骤。

![注册](images/signup.png)

`用户名` 字段将作为您模型名称的一部分（例如 `jmorganca/mymodel`），所以请确保您对所选用户名感到满意。

创建账户并登录后，前往 [Ollama 密钥设置](https://ollama.com/settings/keys) 页面。

按照页面上的指示确定您的 Ollama 公钥的位置。

![Ollama 密钥](images/ollama-keys.png)

点击 `添加 Ollama 公钥` 按钮，并复制粘贴您的 Ollama 公钥内容到文本字段中。

要将模型推送到 [ollama.com](https://ollama.com)，首先确保它正确地使用您的用户名命名。您可能需要使用 `ollama cp` 命令复制您的模型以给它正确的名称。一旦您对模型名称感到满意，使用 `ollama push` 命令将其推送到 [ollama.com](https://ollama.com)。

```shell
ollama cp mymodel myuser/mymodel
ollama push myuser/mymodel
```

一旦您的模型被推送，其他用户可以通过以下命令拉取并运行它：

```shell
ollama run myuser/mymodel
```
