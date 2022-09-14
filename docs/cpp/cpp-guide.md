# C++ 编程指南

## 函数返回大对象应该使用输出参数还是返回值？

对于内置类型(int) 对象的返回，直接使用返回值即可

可以大对象的返回，如果满足 [RVO/NRVO](https://mp.weixin.qq.com/s/LwnDtK6HNZo_StIxQ5yJhA) 优化条件的，可以使用返回值；但是这些优化依赖于编译器，具有不确定性，`建议还是使用输出参数`

## Reference

- [Google开源项目风格指南](https://mp.weixin.qq.com/s/LwnDtK6HNZo_StIxQ5yJhA)