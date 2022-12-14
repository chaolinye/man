# 研发工作优秀实践

## 用清单记录任务

工作中经常会出现多件事情并行的情况，而且很多事情存在需要等待的节点。

用清单记录这些事情，避免遗漏，安排优先级，定期回顾，更好地掌握进度和完成任务，也让自己对工作更有掌握感

> 提供几个清单工具作为参考: [双链笔记本 logseq](https://logseq.com/), ediary, OneNote，Microsoft TODO等

## 经常写文档

在每天完成工作后，思考下今天的工作，是否需要写个文档，以便后续自己查阅（好记性不如烂笔头），也可以提供给他人学习（这样就不用自己亲自指导，会节省很多时间）

> 文档最好是在线文档，比如 `wiki`、`博客` 这类形式，当然公司内容有这样的平台最好，如果没有，可以自己搭建一个文档平台，具体请参考 [《构建文档平台》](../devops/document) 

## 每天看 MR

每天抽写时间来看自己参与的项目代码仓中的每个 MR，这样可以让你对整个项目会更加了解，也学习到别人的优秀代码或者提前发现一些问题

> 坚持这个习惯，可以让你有很大概率可以成为这个项目的负责人

## 接口开发审查

- 是否每个参数都做了校验？

    > string 类型字段至少要做长度校验，最好有正则校验

- 是否充分考虑了边界条件？（鲁棒性）

- 是否考虑了故障情况？（容错性）

- 是否存在跨目录攻击、sql 攻击、横向越权的可能？(安全性)

- 性能能否再优化？（性能）
