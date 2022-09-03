# Java awesome

记录 Java 生态中常用的库和项目

## 常用工具库 - Hutool

[文档](https://hutool.cn/docs/#/)

包含常用的大部分工具方法，日常工作中遇到需要工具类的时候，先看看 hutool 有没有

## 重试库

一个重试库的设计要点：

- 遇到什么异常或者返回需要进行重试？
- 重试的次数或者超时时间？无限重试？
- 重试的方式：立即重试、延时一段时间重试
- 支持重试失败回调，让使用方决定后续的处理
- 支持每次重试失败后的回调，便于使用方记录日志

### Spring Retry

[Github](https://github.com/spring-projects/spring-retry)
[使用参考](https://www.baeldung.com/spring-retry)

### Guava Retry

[Github](https://github.com/rholder/guava-retrying)
[使用参考](https://mp.weixin.qq.com/s/_12vgPi2sDQ6sjJeXLKr4A)

