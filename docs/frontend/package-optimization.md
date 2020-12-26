# 打包优化

## lodash 只打包需要的函数

使用 lodash 的 babel 插件 `babel-plugin-lodash` 可以只打包用到的函数

比如

```js
import _ from 'lodash'
import { add } from 'lodash/fp'

const addOne = add(1)
_.map([1, 2, 3], addOne)
```

`babel-plugin-lodash` 编译后就是

```js
import _add from 'lodash/fp/add'
import _map from 'lodash/map'

const addOne = _add(1)
_map([1, 2, 3], addOne)
```

> 本质上就是 lodash 把每个函数都独立成一个文件，通过 babel 插件在编译的时候把 import 的路径改成对应函数文件路径，之后的打包中就只会引入用到的函数文件

[babel-plugin-lodash 文档](https://github.com/lodash/babel-plugin-lodash)