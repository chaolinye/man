# py-spy 详细使用指南

py-spy 是一个强大的 Python 性能分析工具，最大的优点是**不需要修改代码**且**可以分析运行中的生产环境进程**。

## 1. 安装

```bash
# 基本安装
pip install py-spy
```

## 2. 基本使用

### 实时监控模式 (top)
```bash
# 监控指定 PID 的进程
py-spy top --pid 12345

# 监控 Python 脚本
py-spy top -- python my_script.py

# 带时间戳输出
py-spy top --pid 12345 --timestamp

# 每秒采样 100 次
py-spy top --pid 12345 --rate 100

# 显示本地变量
py-spy top --pid 12345 --locals

# 输出到文件
py-spy top --pid 12345 --output profile.txt
```

### 记录模式 (record)
```bash
# 记录性能数据并生成火焰图
py-spy record -o profile.svg --pid 12345

# 记录特定时长（30秒）
py-spy record -o profile.svg --duration 30 --pid 12345

# 记录并包含子进程
py-spy record -o profile.svg --subprocesses --pid 12345

# 多种输出格式
py-spy record -o profile.json --format json --pid 12345
py-spy record -o profile.txt --format raw --pid 12345
py-spy record -o profile.html --format speedscope --pid 12345
```

### 转储模式 (dump)
```bash
# 显示当前调用栈
py-spy dump --pid 12345

# 显示所有线程的调用栈
py-spy dump --pid 12345 --threads
```

## 3. 实际应用场景

### 场景 1: 分析运行中的 Web 服务器
```bash
# 找到 Gunicorn 主进程
ps aux | grep gunicorn

# 监控主进程 (假设 PID 为 12345)
py-spy top --pid 12345

# 记录 60 秒性能数据
py-spy record -o gunicorn_profile.svg --duration 60 --pid 12345
```

### 场景 2: 分析 Django 应用
```bash
# 直接运行并分析
py-spy record -o django_profile.svg -- python manage.py runserver

# 或者附加到运行中的进程
py-spy top --pid $(pgrep -f "python manage.py runserver")
```

### 场景 3: 分析 Celery Worker
```bash
# 监控 Celery worker
py-spy top --pid $(pgrep -f "celery worker")

# 记录任务执行过程
py-spy record -o celery_profile.svg --duration 120 --pid $(pgrep -f "celery worker")
```

## 4. 高级功能

### 函数过滤
```bash
# 只显示包含 "request" 的函数
py-spy top --pid 12345 --function "request"

# 排除包含 "test" 的函数
py-spy top --pid 12345 --gil --function "request" --exclude "test"
```

### 分析多进程应用
```bash
# 分析整个进程组
py-spy record -o multi_process.svg --subprocesses --pid 12345

# 分析所有 Python 进程
py-spy top --pid $(pgrep -d, python)
```

### 内存分析
```bash
# 显示内存分配（需要 --memory 标志编译）
py-spy record --memory -o memory_profile.svg --pid 12345
```

## 5. 完整实战示例

### 示例应用代码
```python
# app.py
import time
import requests
from concurrent.futures import ThreadPoolExecutor

def slow_calculation(n):
    """模拟慢计算"""
    result = 0
    for i in range(n * 1000):
        result += i * i
    return result

def make_http_request():
    """模拟 HTTP 请求"""
    try:
        response = requests.get('https://httpbin.org/delay/1', timeout=5)
        return response.status_code
    except:
        return None

def process_data():
    """数据处理函数"""
    # CPU 密集型任务
    calculations = [slow_calculation(i) for i in range(100)]
    
    # I/O 密集型任务
    with ThreadPoolExecutor(max_workers=5) as executor:
        results = list(executor.map(lambda x: make_http_request(), range(10)))
    
    return calculations, results

if __name__ == "__main__":
    while True:
        print("Processing...")
        process_data()
        time.sleep(2)
```

### 使用 py-spy 分析

**终端 1 - 运行应用:**
```bash
python app.py
```

**终端 2 - 实时监控:**
```bash
# 找到进程 ID
ps aux | grep "python app.py"

# 开始监控（假设 PID 为 67890）
py-spy top --pid 67890 --rate 50 --duration 30
```

**终端 3 - 生成火焰图:**
```bash
# 记录 60 秒数据生成火焰图
py-spy record -o app_flamegraph.svg --duration 60 --pid 67890

# 生成交互式 HTML 报告
py-spy record -o app_speedscope.html --format speedscope --duration 60 --pid 67890
```

## 6. 输出解读和分析

### top 输出示例:
```
Collecting samples from 'python app.py' (pid: 67890)
Total Samples: 1500
GIL: 45.3% Active, 54.7% Held

%Own   %Total  Function
15.2%  15.2%   slow_calculation
12.1%  12.1%   <listcomp>
8.3%   8.3%    make_http_request
6.7%   6.7%    process_data
```

### 火焰图解读:
- **宽度**表示函数在采样中出现的频率（时间占比）
- **从上到下**显示调用栈关系
- **较宽的顶层函数**可能是性能瓶颈

## 7. 生产环境最佳实践

### 安全考虑
```bash
# 需要 ptrace 权限（Linux）
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope

# 或者使用 sudo
sudo py-spy top --pid 12345
```

### Docker 环境使用
```bash
# 在容器内运行
docker exec -it container_name pip install py-spy
docker exec -it container_name py-spy top --pid 1

# 或者从主机分析
py-spy top --pid $(pgrep -f "python")

# 在 Dockerfile 中安装
RUN pip install py-spy
```

### Kubernetes 环境
```bash
# 进入 pod 分析
kubectl exec -it pod-name -- bash
pip install py-spy
py-spy top --pid 1

# 或者使用临时调试容器
kubectl debug pod/pod-name -it --image=ubuntu --target=app
```

## 8. 与其他工具集成

### 与 perf 结合
```bash
# 使用 perf 进行系统级分析
perf record -g -p 12345
perf script | stackcollapse-perf.pl | flamegraph.pl > perf.svg

# 同时使用 py-spy 进行 Python 级分析
py-spy record -o py_flamegraph.svg --pid 12345
```

### 自动化分析脚本
```bash
#!/bin/bash
# monitor_python.sh

PID=$1
DURATION=${2:-60}

echo "Starting py-spy monitoring for PID: $PID for $DURATION seconds"

# 生成火焰图
py-spy record -o "profile_$(date +%Y%m%d_%H%M%S).svg" \
  --duration $DURATION \
  --pid $PID \
  --subprocesses

echo "Profile saved. Generating top output..."
py-spy top --pid $PID --duration 10 > "top_$(date +%Y%m%d_%H%M%S).txt"
```

## 9. 常见问题排查

### 权限问题
```bash
# 错误: "Permission denied. Try running with sudo"
sudo py-spy top --pid 12345

# 或者临时修改系统设置
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
```

### 进程找不到
```bash
# 确认进程存在且是 Python 进程
ps -p 12345 -o pid,comm,cmd

# 列出所有 Python 进程
pgrep -l python
```

### 符号信息缺失
```bash
# 确保调试符号可用
python -g my_script.py

# 或者使用 --nonblocking 模式
py-spy record --nonblocking -o profile.svg --pid 12345
```

py-spy 的优势在于它的低开销和对生产环境的友好性，是诊断线上 Python 应用性能问题的利器。