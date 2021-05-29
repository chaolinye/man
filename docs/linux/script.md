# 常用脚本

## 获取脚本当前路径

```bash
dir=$(dirname $0)
cwd=$(cd $dir && pwd)
echo $cwd
```