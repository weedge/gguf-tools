# GGUF 工具

这是一个用于操作 GGUF 文件的工具库，正在进行中。虽然该库旨在提供有用的功能，但主要目标之一是提供易于访问的代码库，作为副作用来记录由令人惊叹的 [llama.cpp](https://github.com/ggerganov/llama.cpp) 项目使用的 GGUF 文件：GGUF 文件在本地机器学习领域的使用越来越普遍和核心，因此拥有多个解析器和文件生成器的实现可能非常有用。

名为 **gguf-tools** 的程序使用该库来实现有用和无用的功能，以展示在现实世界中使用该库。目前，该实用程序实现了以下子命令：

### gguf-tools show file.gguf

显示有关 GGUF 文件的详细信息。这将包括所有键值对，包括数组和详细的张量信息。张量的偏移量将相对于文件的开头（因此它们实际上是绝对偏移量），而不是像 GGUF 格式中的数据部分的开头。

示例输出：

```
./gguf-tools show models/phi-2.Q8_0.gguf | head -20        :main*: ??
models/phi-2.Q8_0.gguf (ver 3): 20 key-value pairs, 325 tensors
general.architecture: [string] phi2
general.name: [string] Phi2
phi2.context_length: [uint32] 2048
phi2.embedding_length: [uint32] 2560
phi2.feed_forward_length: [uint32] 10240
phi2.block_count: [uint32] 32
phi2.attention.head_count: [uint32] 32
phi2.attention.head_count_kv: [uint32] 32
phi2.attention.layer_norm_epsilon: [float32] 0.000010
phi2.rope.dimension_count: [uint32] 32
general.file_type: [uint32] 7
tokenizer.ggml.add_bos_token: [bool] false
tokenizer.ggml.model: [string] gpt2
tokenizer.ggml.tokens: [array] [!, ", #, $, %, &, ', (, ), *, +, ,, -, ., /, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, :, ;, <, =, >, ... 51170 more items of 51200]

... 更多键值对 ...

q8_0 tensor token_embd.weight @1806176, 131072000 weights, 139264000 bytes
f32 tensor blk.0.attn_norm.bias @141070176, 2560 weights, 10240 bytes
f32 tensor blk.0.attn_norm.weight @141080416, 2560 weights, 10240 bytes
f32 tensor blk.0.attn_qkv.bias @141090656, 7680 weights, 30720 bytes
q8_0 tensor blk.0.attn_qkv.weight @141121376, 19660800 weights, 20889600 bytes
f32 tensor blk.0.attn_output.bias @162010976, 2560 weights, 10240 bytes
q8_0 tensor blk.0.attn_output.weight @162021216, 6553600 weights, 6963200 bytes
f32 tensor blk.0.ffn_up.bias @168984416, 10240 weights, 40960 bytes

... 更多张量 ...
```

### gguf-tools compare file1.gguf file2.gguf

此工具有助于了解两个 LLM（或其他以 GGUF 文件形式分发的模型）是否相关，例如一个是否是另一个的微调，或者两者是否都是从同一个父模型进行微调的。

对于每个匹配的张量（名称和参数数量相同），该命令计算平均权重差异（以百分比表示，因此在 -N 到 +N 范围内的随机分布平均上与另一个随机分布相比较将达到 100% 差异）。这有助于查看模型是否是另一个模型的微调版本，它被微调了多少，哪些层在微调时被冻结等。请注意，由于量化，即使功能上等效的张量可能也存在一些小的平均差异。

示例输出：

```
./gguf-tools compare mistral-7b-instruct-v0.2.Q8_0.gguf \
                     solar-10.7b-instruct-v1.0-uncensored.Q8_0.gguf
[token_embd.weight]: avg weights difference: 44.539944%
[blk.0.attn_q.weight]: avg weights difference: 48.717736%
[blk.0.attn_k.weight]: avg weights difference: 56.201885%
[blk.0.attn_v.weight]: avg weights difference: 47.087249%
[blk.0.attn_output.weight]: avg weights difference: 47.663048%
[blk.0.ffn_gate.weight]: avg weights difference: 37.508761%
[blk.0.ffn_up.weight]: avg weights difference: 39.061584%
[blk.0.ffn_down.weight]: avg weights difference: 39.632648%
...
```

### gguf-tools inspect-tensor file.gguf tensor.name [count]

显示指定张量的所有权重值（如果未指定 count，则仅显示前 count 个）。这对于低级别的任务很有用，比如检查量化是否按预期工作，查看引入的误差，模型指纹等。

### gguf-tools split-mixtral 65230776370407150546470161412165 mixtral.gguf out.gguf

从 Mixtral 7B MoE 中提取一个 7B 模型 `out.gguf`，使用指定的每层 MoE ID（在序列 652... 中有 32 个数字）。

请注意，通过 split-mixtral 方式获得的模型实际上没有执行任何有用的工作。这只是一个实验和一个不太重要的任务，用来展示如何使用该库。很可能很快就会被移除，一旦我有更有趣和有用的示例要展示，比如模型合并。

## GGUF 库 API

目前唯一的文档是实现本身：请参阅 gguf-tools.c 以获取使用信息。这可能会在以后改变，但目前该库正在积极开发中。

代码有很好的注释，到目前为止，API 极其简单易懂且易于使用。

## 限制

许多量化格式尚未支持。

## 规范文档

* [官方 GGUF 规范](https://github.com/ggerganov/ggml/blob/master/docs/gguf.md)，描述了文件布局和元数据。
* [用于量化 GGUF 模型的量化格式](https://github.com/ggerganov/ggml/blob/master/src/ggml-quants.h)。