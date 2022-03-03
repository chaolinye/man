# 性能监测

## 线程

- 查看占用 CPU 最高的线程

    ```bash
    # 根据关键字查找进程
    ps aux | grep <keyword>

    # 查看进程中有哪些进程
    ps -T -p <pid>

    # 查看占用 CPU 最高的线程
    top -H -p <pid>

    # 线程 id 转十六进制
    printf '%x\n' <threadId>

    # 查看线程栈
    jstack -l <pid> | grep <16进制 threadId> -A 20
    ```

## QPS

测试工具有 ab

### Apache Benchmark

> apache 旗下的 http server 的性能评测工具，简称 ab

[官方文档](https://httpd.apache.org/docs/2.4/programs/ab.html)

- 安装

    ```bash
    # centos
    yum -y install httpd-tools

    # ubuntu
    apt-get install apache2-utils
    ```

- 基础使用

    ```bash
    # 10 并发，1000 个请求
    ab -c10 -n1000 https://www.baidu.com/
    ```

结果:

> 重点指标：`Requests per second`(QPS), `百分比(90%, 95%, 99%)耗时`

> 指标中有两个 `Time per request`, 第一个是请求平均耗时（每个请求开始到结束的时间的平均值），第二个是测试时间除以完成的请求数即 `Time taken for tests/Complete requests`

```bash
This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking www.baidu.com (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests


Server Software:        BWS/1.1
Server Hostname:        www.baidu.com
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES128-GCM-SHA256,2048,128
Server Temp Key:        ECDH P-256 256 bits
TLS Server Name:        www.baidu.com

Document Path:          /
Document Length:        227 bytes

Concurrency Level:      10
Time taken for tests:   24.836 seconds
Complete requests:      1000
Failed requests:        0
Total transferred:      1110891 bytes
HTML transferred:       227000 bytes
Requests per second:    40.26 [#/sec] (mean)
Time per request:       248.358 [ms] (mean)
Time per request:       24.836 [ms] (mean, across all concurrent requests)
Transfer rate:          43.68 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:      158  182  28.6    177     520
Processing:    49   62  10.9     60     213
Waiting:       49   61  10.6     59     213
Total:        211  244  33.3    236     586

Percentage of the requests served within a certain time (ms)
  50%    236
  66%    240
  75%    243
  80%    246
  90%    259
  95%    306
  98%    345
  99%    405
 100%    586 (longest request)
```

- 真实的例子

    ```bash
    ab -c10 -n100 \
        -H 'Connection: keep-alive' \
        -H 'Cache-Control: no-cache' \
        -p data.json
        -T 'application/json' \
        https://www.baidu.com
    ```

    > 对于浏览器中的请求，可以从 `F12 开发者工具 -> Network -> 选中一个右击 -> copy -> copy as curl` 获取到 curl 格式链接，再改造成 ab 格式即可

    ### JMeter

    > ab 只能单机测试，无法模拟真实的多客户端场景，比较适合于开发人员快速验证 QPS

    JMeter 是一个专业的压测工具

    [官方文档](https://jmeter.apache.org/)

    教程

    - [JMeter 入门教程](https://www.cnblogs.com/tankxiao/p/4045439.html)
    - [Jmeter 教程 简单的压力测试](https://www.cnblogs.com/TankXiao/p/4059378.html)

    ## References

    - [Rails Apache Benchmark 的使用的个人浅薄经验](https://ruby-china.org/topics/13870)