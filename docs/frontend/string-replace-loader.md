# string-replace-loader 修复三方库 bug

## 问题

`html2canvas` 截取无法显示跨域且需要 cookie 认证的图片

## 原因

`html2canvas` 在图片跨域情况下写死了 `crossOrigin` 为 `anonymous`

> `crossOrigin="anonymous"` 表示该元素的 CORS 请求将不设置凭据标志。即无法传递 cookie  
> `crossOrigin="use-credentials"` 表示该元素的 CORS请求将设置凭证标志；即会传递 cookie

```js
private async loadImage(key: string) {
    const isSameOrigin = CacheStorage.isSameOrigin(key);
    const useCORS =
        !isInlineImage(key) && this._options.useCORS === true && FEATURES.SUPPORT_CORS_IMAGES && !isSameOrigin;
    const useProxy =
        !isInlineImage(key) &&
        !isSameOrigin &&
        typeof this._options.proxy === 'string' &&
        FEATURES.SUPPORT_CORS_XHR &&
        !useCORS;
    if (!isSameOrigin && this._options.allowTaint === false && !isInlineImage(key) && !useProxy && !useCORS) {
        return;
    }

    let src = key;
    if (useProxy) {
        src = await this.proxy(src);
    }

    Logger.getInstance(this.id).debug(`Added image ${key.substring(0, 256)}`);

    return await new Promise((resolve, reject) => {
        const img = new Image();
        img.onload = () => resolve(img);
        img.onerror = reject;
        //ios safari 10.3 taints canvas with data urls unless crossOrigin is set to anonymous
        if (isInlineBase64Image(src) || useCORS) {
            img.crossOrigin = 'anonymous';
        }
        img.src = src;
        if (img.complete === true) {
            // Inline XML images may fail to parse, throwing an Error later on
            setTimeout(() => resolve(img), 500);
        }
        if (this._options.imageTimeout > 0) {
            setTimeout(
                () => reject(`Timed out (${this._options.imageTimeout}ms) loading image`),
                this._options.imageTimeout
            );
        }
    });
}
```

## 解决方法

由于 `html2canvas` github 上已有相应的 issue，但是短期内难以修复

而自己维护一个 `html2canvas` 的 fork 库来修改又比较麻烦，且不一定符合公司安全规范

最后通过 `string-replace-loader` 在打包过程中将 `anonymous` 替换成 `use-credentials` 暂时修复了这个问题

```js
// vue.config.js
module.exports = {
    chainWebpack: config => {
        config.module
        .rule('html2canvas')
        .test(/html2canvas.*\.js$/)
        .use('string-replace-loader')
            .loader('string-replace-loader')
            .options({
                search: "img.crossOrigin = 'anonymous'",
                replace: "img.crossOrigin = 'use-credentials'"
            })
            .end();
    }
}
```