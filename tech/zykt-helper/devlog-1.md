---
title: 智云课堂辅助脚本开发日志（一）：启动
sync: /tech/zykt-helper/devlog-1.md
---


作为现代的大学生，自然是非常喜欢使用智云课堂。

However，作为，多少有点不好用的地方。

本着“求人不如求己”的原则，在使用了智云课堂一个学期之后，我决定还是自己上手开发一个智云课堂的辅助脚本。

## 技术选型

考虑到智云课堂的使用场景，这款辅助脚本并不需要特别“轻量”，而应该把该做的功能都做了，同时还有较高的可维护性（方便传宗接代给之后的小朋友维护）。

思量一番后还是考虑选用之前试过的 webpack 方案。我的博客曾经有一篇文章怎么用 webpack 配合 TamperMonkey 开发用户脚本，但后来觉得写的过于垃圾就删掉了==

首先按照一般方式创建一个 webpack 项目，npm 装包的过程就省略了，请自行参考我的 package.json。

填入 webpack 配置如下：

```js
// ./webpack.config.js
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: ['babel-loader'],
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
};
```

