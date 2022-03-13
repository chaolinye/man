# POJO 映射

基于当前 Java Web 后端的编码规范，往往会存在大量包含相同字段的 POJO 类（VO，DTO，PO 等）。

这些 POJO 类往往需要互相转换，大量的 set 语句繁琐且容易遗漏。

这时我们可以使用 POJO 映射工具库来完成这个转换，常见的转换工具库可以分为三类

## JSON 序列化和反序列化

具体实现请参考前面的章节 [深拷贝](./deep-copy)

缺点： 性能差，不支持字段名称不一致（通过JSON注解可以解决，但是侵入性太强）

## 反射 -- BeanUtils

优点：简洁，通过封装工具类 `ConvertBeanUtil` 可以更简洁
缺点： 不支持字段名称不一致

## 编译时注解处理器 -- MapStruct

缺点：需要定义额外的 Mapper 类

## 比较

性能方面: `MapStruct` > `BeanUtils` > `JSON`

编码简洁方面: `BeanUtils` > `JSON` > `MapStruct`

建议优先使用 `BeanUtils`, 包含子对象使用 `JSON`, 存在大量不一致字段使用 `MapStruct`

## References

- [Quick Guide to MapStruct](https://www.baeldung.com/mapstruct)
- [MapStruct 官网](https://mapstruct.org/)