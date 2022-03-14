#  require.context 导入文件夹

前端开发中，经常需要用一个入口文件，把当前文件夹下的其他文件 export，那么每加一个文件，都需要修改入口文件，这违背开闭原则

## webpack require.context

webpack 支持通过 require.context 导入多个模块

本质上是生成一个模块的映射，require.context 的返回：


```json
{
  "./table.ejs": 42,
  "./table-row.ejs": 43,
  "./directory/another.ejs": 44
}
```

> 这也意味者，所有的文件都会被 webpack 打包进来

入口文件读取子模块内容：

```js
const cache = {};

function importAll(r) {
  r.keys().forEach((key) => (cache[key] = r(key)));
}

importAll(require.context('../components/', true, /\.js$/));

export default cache;
```

## References

- [webpack require.context 文档](https://webpack.docschina.org/guides/dependency-management/)
- [使用require.context实现前端工程自动化](https://www.jianshu.com/p/c894ea00dfec)

