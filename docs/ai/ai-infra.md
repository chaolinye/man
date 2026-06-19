# AI Infra

## 模型下载

[Hugging Face](https://huggingface.co/): 模型仓库

模型下载命令:

```
# 设置国内镜像
export HF_ENDPOINT=https://hf-mirror.com
# 解决证书错误（可选）
pip install certifi
export SSL_CERT_FILE=$(python -c "import certifi; print(certifi.where())")
export REQUESTS_CA_BUNDLE=$(python -c "import certifi; print(certifi.where())")
# 使用 hf 命令行工具下载
hf download hf://unsloth/Qwen3.6-35B-A3B-MTP-GGUF/Qwen3.6-35B-A3B-UD-Q4_K_M.gguf
```

## 模型部署

### llama.cpp 安装

预编译版本可以从 Llama.cpp Github [Release](https://github.com/ggml-org/llama.cpp/releases) 页获取。

> 注意，CUDA 版本要选择下载后面带 CUDA xx.x DDLs 的版本

如果要使用一些主线还没有待合入的新特性，那就需要本地编译 Llama.cpp 源码。本地编译可以参考[这篇博客](https://zhuanlan.zhihu.com/p/2038651608892953654)。注意点：

1. 博客中的是 Visual Studio 2022， 现在最新的是 Visual Stuido 2026， 直接下载最新版本（组件中不勾选`MSVC v143 - VS 2022 C++ x64/x86 生成工具`也可以）。
2. 要先安装 Visual Studio，再安装 CUDA Toolkit。因为 CUDA Toolkit 安装时会根据本地是否安装了 Visual Studio 决定是否安装对应的连接器。将 CUDA Toolkit 安装目录的 `bin` 子目录添加到 PATH 变量中，这样后续编译时就不需要指定 CUDA 的路径，比较方便。
3. Visual Studio 2026 对应的命令，要在 `Developer PowerShell for VS` 或者 `Developer Command Prompt for VS` 终端运行： 
  ```bash
  # 生成编译图纸， 指定 Visual Studio 2026
  cmake -B build -G "Visual Studio 18 2026" -A x64 -DGGML_CUDA=ON
  # 用 CPU 多核加速编译过程，-j 后面的数字可以是你的 CPU 核心数
  cmake --build build --config Release -j 10
  ```

### llama.cpp 运行模型

```bash
# 50 tok/s
./build/bin/Release/llama-server.exe -m /d/Data/models/Qwen3.6-35B-A3B-UD-IQ3_S.gguf \
  --ctx-size 131072 \
  --batch-size 1024 \
  --ubatch-size 512 \
  --cache-reuse 256 \
  --cache-ram 32768 \
  --n-gpu-layers 99 \
  --threads 16 \
  --cache-type-k q4_0 \
  --cache-type-v q4_0 \
  --flash-attn on \
  --mlock \
  --temp 0.7 \
  --top-p 0.8 \
  --top-k 20 \
  --min-p 0.05 \
  --reasoning off \
  --port 8080 \
  --host 0.0.0.0

# 20 tok/s
./build/bin/Release/llama-server.exe -m /d/Data/models/Qwen3.6-27B-NEO-CODE-HERE-2T-OT-IQ3_M.gguf \
  --ctx-size 131072 \
  --batch-size 1024 \
  --ubatch-size 512 \
  --cache-ram 32768 \
  --n-gpu-layers 99 \
  --threads 16 \
  --cache-type-k q4_0 \
  --cache-type-v q4_0 \
  --flash-attn on \
  --mlock \
  --port 8080 \
  --host 0.0.0.0
```

## References

- [AI Infra](https://infrasys-ai.github.io/aiinfra-docs/index.html)
- [minimind](https://github.com/jingyaogong/minimind)
- [从零构建大模型](https://book.douban.com/subject/37305124/)
