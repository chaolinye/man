# Skills

## Skill 撰写指南（How to write great skill）

what makes a skill great

### Trigger（触发）

Decide if your skill is user-invoked or model-invoked?

Model-invoked 的缺点：
1. Context Load，Description 占用上下文
2. 有概率调用不到

> context pointer：description 就是 skill 在 llm context 中的 pointer

User-invoked 的缺点：
1. 认知负载重。需要深入理解Skill，才能最大限度地使用

> 对于技术人员，推荐 User-invoked，更好地掌握 skill

### Structure（结构）

- Steps: the step-by-step procedure
- Reference: supporting information

Tips: 
1. structure skills into steps and reference
2. make SKILL.md as small as possible: SKILL.md 越精简，更容易维护、更容易审核、更节省token
3. branches: Different ways skill can be used。
    > 如果某个 reference 只在某个分支中用到，就可以从 SKILL.md 中移出去，称为外部引用 
4. Hide branch reference material behind context pointers

### Steering（控制）

Failure Mode: The agent doesn't do what I want 

Tip: Use Leading Word(引导词: 较小字数的短语， 在较小的空间内蕴含丰富的含义)
1. Reasoning Tokens
2. Changes Behavior

原理：skill 中将大量信息浓缩到 leading word，重复多次，agent 自己也会不断重复这个 leading word，这个 leading word 的遵循度会很高。可以类比于 TF-ITF，leading word 的重要性很高，是关键特征，agent 输出的时候对于 leading word 也会更加的尊重

例子：
- Problem: Agent Code Layer-by-Layer
- Target: 像人一样先实现一个可用模块，再扩展
- leading Word: Vertical Slice

Tip: Use consistent leading words

Failure Mode: The agent didn't do enougth legwork 

- 例子：Plan Mode has two steps: 1. ask clarifying questions, 2. create a plan. 
- 问题：agent 急于开始第2步的创建计划，对于第一步的Ask投入不充分
- 方案：拆成两个skill

Tip: Increase legwork by hiding future steps

### Pruning（精简）

Failure Mode: Massive Skills(过于庞大)

Tip: Don't repeat yourself. 

> 保证真理的唯一性。例如同一个 reference 不应该在多个skill重复 

Failure Mode: Sediment(沉积，不断增加，不敢删除)

> 某个分支的内容移到外部引用，无关内容删除

Failure Mode: No-ops。某些内容存在是否，不影响Agent的行为，也就是Agent的通用知识能cover这些内容

Tip: Use the deletion test. 删除试试Agent行为是否受到影响



## Design Skills

[程序员必备的 Design Skills](https://mp.weixin.qq.com/s/BkReBmLwipxyYNfU-UmyHw?token=1118830042&lang=zh_CN):

![](../images/design-skills.png)

## Coding Skills

### 代码阅读

[codebase-to-course](https://github.com/zarazhangrui/codebase-to-course)

### 文档

[ppt-master](https://github.com/hugohe3/ppt-master): ppt 生成器。需要 image 模型生成配图（可以用 MiniMax 的 image-01)

## References

- [Skills](http://github.com/vercel-labs/skills#readme): Skill 包管理器
