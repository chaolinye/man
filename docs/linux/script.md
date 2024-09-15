# 常用脚本

## 获取脚本当前路径

```bash
dir=$(dirname $0)
cwd=$(cd $dir && pwd)
echo $cwd
```

## 优雅地处理错误

```bash
trap 'echo "Error occurred"; cleanup; exit 1' ERR

function cleanup() {
    # Cleanup code
}
```

## 日志记录

```bash
logfile="script.log"
exec > >(tee -i $logfile)
exec 2>&1

echo "Script started"
```

## Refercense

- [Bash 脚本高级技巧](https://omid.dev/2024/06/19/advanced-shell-scripting-techniques-automating-complex-tasks-with-bash/)