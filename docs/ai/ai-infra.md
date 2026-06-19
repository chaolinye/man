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

## References

- [AI Infra](https://infrasys-ai.github.io/aiinfra-docs/index.html)
- [minimind](https://github.com/jingyaogong/minimind)
- [从零构建大模型](https://book.douban.com/subject/37305124/)
