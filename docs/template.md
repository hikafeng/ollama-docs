# 模板

Ollama 提供了一个强大的模板引擎，基于 Go 内置的模板引擎，用于为你的大型语言模型构建提示。这一功能是充分利用你的模型的重要工具。

## 基本模板结构

一个基本的 Go 模板由三个主要部分组成：

* **布局**：模板的整体结构。
* **变量**：动态数据的占位符，在模板渲染时将被替换为实际值。
* **函数**：可用于操作模板内容的自定义函数或逻辑。

以下是一个简单的聊天模板示例：

```gotmpl
{{- range .Messages }}
{{ .Role }}: {{ .Content }}
{{- end }}
```

在这个例子中，我们有：

* 基本的消息结构（布局）
* 三个变量：`Messages`、`Role` 和 `Content`（变量）
* 一个遍历项目数组的自定义函数（操作）（`range .Messages`）并显示每一项

## 将模板添加到你的模型中

默认情况下，导入到 Ollama 的模型具有默认模板 `{{ .Prompt }}`，即用户输入将直接传递给 LLM。这对于文本或代码补全模型来说是合适的，但缺少聊天或指令模型的重要标记。

在这些模型中省略模板会将正确模板化输入的责任交给用户。添加模板可以让用户轻松地从模型中获得最佳结果。

要在你的模型中添加模板，你需要在 Modelfile 中添加 `TEMPLATE` 命令。以下是使用 Meta 的 Llama 3 的示例。

```dockerfile
FROM llama3.2

TEMPLATE """{{- if .System }}<|start_header_id|>system<|end_header_id|>

{{ .System }}<|eot_id|>
{{- end }}
{{- range .Messages }}<|start_header_id|>{{ .Role }}<|end_header_id|>

{{ .Content }}<|eot_id|>
{{- end }}<|start_header_id|>assistant<|end_header_id|>

"""
```

## 变量

`System`（字符串）：系统提示

`Prompt`（字符串）：用户提示

`Response`（字符串）：助手响应

`Suffix`（字符串）：插入在助手响应后的文本

`Messages`（列表）：消息列表

`Messages[].Role`（字符串）：角色，可以是 `system`、`user`、`assistant` 或 `tool` 之一

`Messages[].Content`（字符串）：消息内容

`Messages[].ToolCalls`（列表）：模型想要调用的工具列表

`Messages[].ToolCalls[].Function`（对象）：要调用的函数

`Messages[].ToolCalls[].Function.Name`（字符串）：函数名称

`Messages[].ToolCalls[].Function.Arguments`（映射）：参数名称到参数值的映射

`Tools`（列表）：模型可以访问的工具列表

`Tools[].Type`（字符串）：模式类型。`type` 始终为 `function`

`Tools[].Function`（对象）：函数定义

`Tools[].Function.Name`（字符串）：函数名称

`Tools[].Function.Description`（字符串）：函数描述

`Tools[].Function.Parameters`（对象）：函数参数

`Tools[].Function.Parameters.Type`（字符串）：模式类型。`type` 始终为 `object`

`Tools[].Function.Parameters.Required`（列表）：必需属性列表

`Tools[].Function.Parameters.Properties`（映射）：属性名称到属性定义的映射

`Tools[].Function.Parameters.Properties[].Type`（字符串）：属性类型

`Tools[].Function.Parameters.Properties[].Description`（字符串）：属性描述

`Tools[].Function.Parameters.Properties[].Enum`（列表）：有效值列表

## 提示和最佳实践

使用 Go 模板时，请记住以下提示和最佳实践：

* **注意点符号**：控制流结构如 `range` 和 `with` 会改变 `.` 的值
* **超出范围的变量**：使用 `$.` 引用当前不在范围内的变量，从根开始
* **空白控制**：使用 `-` 来修剪前导（`{{-`）和尾随（`-}}`）空白

## 示例

### 消息示例

#### ChatML

ChatML 是一种流行的模板格式。它可用于 Databrick 的 DBRX、Intel 的 Neural Chat 和 Microsoft 的 Orca 2 等模型。

```go
{{- range .Messages }}<|im_start|>{{ .Role }}
{{ .Content }}<|im_end|>
{{ end }}<|im_start|>assistant
```

### 工具示例

通过向模板添加 `{{ .Tools }}` 节点，可以为模型添加工具支持。这个功能对于训练调用外部工具的模型很有用，可以成为检索实时数据或执行复杂任务的强大工具。

#### Mistral

Mistral v0.3 和 Mixtral 8x22B 支持工具调用。

```go
{{- range $index, $_ := .Messages }}
{{- if eq .Role "user" }}
{{- if and (le (len (slice $.Messages $index)) 2) $.Tools }}[AVAILABLE_TOOLS] {{ json $.Tools }}[/AVAILABLE_TOOLS]
{{- end }}[INST] {{ if and (eq (len (slice $.Messages $index)) 1) $.System }}{{ $.System }}

{{ end }}{{ .Content }}[/INST]
{{- else if eq .Role "assistant" }}
{{- if .Content }} {{ .Content }}</s>
{{- else if .ToolCalls }}[TOOL_CALLS] [
{{- range .ToolCalls }}{"name": "{{ .Function.Name }}", "arguments": {{ json .Function.Arguments }}}
{{- end }}]</s>
{{- end }}
{{- else if eq .Role "tool" }}[TOOL_RESULTS] {"content": {{ .Content }}}[/TOOL_RESULTS]
{{- end }}
{{- end }}
```

### 填充中间示例

通过向模板添加 `{{ .Suffix }}` 节点，可以为模型添加填充中间支持。这个功能对于训练在用户输入中间生成文本的模型很有用，如代码补全模型。

#### CodeLlama

CodeLlama [7B](https://ollama.com/library/codellama:7b-code) 和 [13B](https://ollama.com/library/codellama:13b-code) 代码补全模型支持填充中间。

```go
<PRE> {{ .Prompt }} <SUF>{{ .Suffix }} <MID>
```

> [!注意]
> CodeLlama 34B 和 70B 代码补全以及所有指令和 Python 微调模型都不支持填充中间。

#### Codestral

Codestral [22B](https://ollama.com/library/codestral:22b) 支持填充中间。

```gotmpl
[SUFFIX]{{ .Suffix }}[PREFIX] {{ .Prompt }}
```
