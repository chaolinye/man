# Vue 事件机制

- [官方文档](https://cn.vuejs.org/v2/guide/events.html)
- [事件机制](https://github.com/answershuto/learnVue/blob/master/docs/Vue%E4%BA%8B%E4%BB%B6%E6%9C%BA%E5%88%B6.MarkDown)

## FAQ

### v-on 的值定义了固定参数，如何获取事件参数

- 问题示例代码：

    ```vue
    <component-a @change="onChange('a')">
    <component-a @change="onChange('b')">
    ```

- 解决方法

    获取第一个事件参数

    ```vue
    <component-a @change="onChange('a', arguments)">
    <component-b @change="onChange('b', arguments)">
    ```

    获取全部事件参数

    ```vue
    <component-a @change="onChange('a', arguments)">
    <component-b @change="onChange('b', arguments)">
    ```

- 源码分析

```js
function genHandler (
  name: string,
  handler: ASTElementHandler | Array<ASTElementHandler>
): string {
  if (!handler) {
    return 'function(){}'
  }

  if (Array.isArray(handler)) {
    return `[${handler.map(handler => genHandler(name, handler)).join(',')}]`
  }

  const isMethodPath = simplePathRE.test(handler.value)
  const isFunctionExpression = fnExpRE.test(handler.value)

  if (!handler.modifiers) {
    return isMethodPath || isFunctionExpression
      ? handler.value
      : `function($event){${handler.value}}` // 如果不是方法名或者函数表达式，会在外面包一层 function, 第一个入参名就是 $event
  } else {
    let code = ''
    let genModifierCode = ''
    const keys = []
    for (const key in handler.modifiers) {
      if (modifierCode[key]) {
        genModifierCode += modifierCode[key]
        // left/right
        if (keyCodes[key]) {
          keys.push(key)
        }
      } else {
        keys.push(key)
      }
    }
    if (keys.length) {
      code += genKeyFilter(keys)
    }
    // Make sure modifiers like prevent and stop get executed after key filtering
    if (genModifierCode) {
      code += genModifierCode
    }
    const handlerCode = isMethodPath
      ? handler.value + '($event)'
      : isFunctionExpression
        ? `(${handler.value})($event)`
        : handler.value
    return `function($event){${code}${handlerCode}}`
  }
}
```