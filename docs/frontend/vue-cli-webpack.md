# vue-cli webpack 配置修改

## vue-cli webpack 配置查看

```bash
# 查看 vue-cli-service serve 的 webpack 配置
vue inspect
# 或者
vue-cli-service inspect

# 查看 vue-cli-service build 的 webpack 配置
vue inspect --mode=production
# 或者
vue-cli-service inspect --mode=production
```

## vue-cli webpack 配置修改

有三种修改方法

- configureWebpack 选项提供一个对象

    ```js
    // vue.config.js
    module.exports = {
    configureWebpack: {
        plugins: [
        new MyAwesomeWebpackPlugin()
        ]
    }
    }
    ```

- configureWebpack 选项提供一个返回对象的函数

    ```js
    // vue.config.js
    module.exports = {
    configureWebpack: config => {
        if (process.env.NODE_ENV === 'production') {
        // 为生产环境修改配置...
        } else {
        // 为开发环境修改配置...
        }
    }
    }
    ```

- chainWebpack 方法对 webpack config 对象进行操作

    ```js
    // vue.config.js
    module.exports = {
    chainWebpack: config => {
        config.module
        .rule('vue')
        .use('vue-loader')
            .tap(options => {
            // 修改它的选项...
            return options
            })
    }
    }
    ```

## References

- [官方文档](https://cli.vuejs.org/zh/guide/webpack.html#%E7%AE%80%E5%8D%95%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%B9%E5%BC%8F)
- [webpack-chain 操作示例](https://juejin.cn/post/6844904138954801166)