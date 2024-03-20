# 工具

## 实现英语读音

参考实现: [earthworm](https://github1s.com/cuixueshe/earthworm/blob/main/apps/client/composables/main/englishSound/index.ts)

```js
const audio = new Audio();
function updateSource(src) {
  audio.src = src;
  audio.load();
}

function play() {
  audio.play()
}

let word = "我不知道"
updateSource(`https://dict.youdao.com/dictvoice?audio=${word}&type=1`);
play();
```