# vue-cli3 中使用骨架屏

## 为什么要使用骨架屏

> 1. 白屏问题一直是前端的痛点，在使用 vue SPA 中，经常会遇到白屏时间过长导致用户体验的差的情况，如何解决这个问题？大部分同学可能会想到优化 webpack、优化网络请求等。实际上，有一部分用户因为网络问题或其他问题，导致了页面速度加载缓慢，等待时间过长，这部分用户可能还没意识到是自己“硬件”问题，他们期待 2 秒内页面加载完成。显然刚才说的优化方案不是理想方案。
> 2. 以往的方案是写一个 loading 动画，比如小菊花，一部分人看到菊花转动超过 3 秒，页面还没出来，会有一定的焦虑感。
> 3. 如果在页面真正解析和应用启动之前，给用户展示页面的大致结构，通过骨架页面的明暗变化，加上一定的 css3 动效，让用户看到页面正在努力加载中，进而提升用户的感知度，留住这部分用户。

## 骨架屏的组成部分

1. 文本块：指定宽高的灰色条纹
2. 图片块：灰色的矩形单元
3. 渐进动效：从左至由的明暗变化动画
4. 圆块：圆块块

## 实现方案

> 1. 为了实现骨架屏的可定制化，需要在全局注册好骨架屏的文本块，图片块，渐进动效和圆形块
> 2. 使用 vue-server-render，将注册好的组件文件在 node 端打包成一个 bundle，然后再由 bundle 生成对应的 html
> 3. vue-server-renderer/server-plugin 是提供给 webpack 读取 bundle 对象的一个插件，在执行构建过程中，会生成了一个 skeleton.json 文件，这个文件存储着骨架屏的样式与内容(类似 AST)
> 4. 客户端创建 BundleRenderer 实例时，会读取到 skeleton.json，将 json 文件还原，替换 index.html 中#app，直到页面加载完毕
> 5. 需要注意的是，骨架屏在加载的时候是 node 端，所以需要设置`target:target: "node"`,`output: {libraryTarget: "commonjs2"}`

## 项目目录

```txt
│  .browserslistrc   # 浏览器兼容
│  .gitignore        # git忽略上传的文件格式
│  babel.config.js   # babel语法编译
│  package.json      # 项目基本信息
│  postcss.config.js # CSS预处理器
│  vue.config.js     # 配置文件
│
├─build
│  └─skeleton
│          skeleton.client.js # 骨架屏客户端
│          skeleton.config.js # 骨架屏服务端参数
│          skeleton.entry.js  # 骨架屏服务端入口
│          skeleton.home.vue  # 骨架屏结构
│          skeleton.json      # 生成的含骨架屏样式与内容的json文件
│          skeleton.server.js # 骨架屏服务端
│          skeleton.template.html # 读入的index.html
│          skeleton.test.vue      # 一个测试模板
│
├─public
│      favicon.ico
│      index.html # 静态index.html，最终会被build/skeleton/skeleton.template.html替换
│
├─screenshot # 截图
│      after-skeleton-dom.png
│      after-skeleton-loading.png
│      skeleton-dom.png
│      skeleton-loading.png
│
└─src
    │  App.vue
    │  main.js
    │  router.js
    │
    ├─assets
    │      banner.png
    │      logo.png
    │
    ├─components
    │  └─skeleton # 骨架屏组件
    │      │  install.js
    │      │  skeleton.less
    │      │  skeleton.vue
    │      │
    │      ├─basic
    │      │      skeleton-circle.vue
    │      │      skeleton-square.vue
    │      │
    │      └─layout
    │              skeleton-column.vue
    │              skeleton-row.vue
    │
    └─views
            home.vue
```

## 执行

```bash
# 第一步：安装依赖
npm install
# 第二步：注入骨架屏
npm run serve
# 第三步：开启服务器
npm run build
# build
npm run build
```

## 具体细节

- node 端

```js
// build/skeleton/skeleton.config.js
const { resolve } = require("path");
const nodeExternals = require("webpack-node-externals");
// webpack中读取bundle对象
const VueSSRServerPlugin = require("vue-server-renderer/server-plugin");
const autoprefixer = require("autoprefixer");
const { VueLoaderPlugin } = require("vue-loader");

// 依赖的文件
const skeletonEntry = resolve(__dirname, "skeleton.entry.js");

module.exports = {
  mode: "production",
  target: "node",
  entry: {
    skeleton: skeletonEntry
  },
  output: {
    path: resolve(__dirname, "."),
    filename: "[name].js",
    libraryTarget: "commonjs2"
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["vue-style-loader", "css-loader", "postcss-loader"]
      },
      {
        test: /\.less$/,
        use: ["vue-style-loader", "css-loader", "less-loader", "postcss-loader"]
      },
      {
        test: /\.vue$/,
        use: [
          {
            loader: "vue-loader",
            options: {
              loaders: {
                scss: ["vue-style-loader", "css-loader", "less-loader"]
              },
              postcss: [autoprefixer()]
            }
          }
        ]
      }
    ]
  },
  // 防止将某些 import 的包打包到 bundle 中，只在运行时从外部获取这些扩展依赖
  externals: nodeExternals({
    whilelist: /\.css$/
  }),
  resolve: {
    extensions: [".js", ".vue", ".json"],
    alias: {
      components: resolve("src/components")
    }
  },
  plugins: [
    new VueLoaderPlugin(),
    // 指定输出的json文件名，执行构建过程中，生成了一个skeleton.json文件，
    // 这个文件存储着骨架屏的样式与内容，在创建BundleRenderer 实例，会读取到
    new VueSSRServerPlugin({
      filename: "skeleton.json"
    })
  ]
};
```

- 客户端

```js
// build/skeleton/skeleton.client.js
const fs = require("fs");
const { resolve } = require("path");
const htmlMinifier = require("html-minifier");
const chalk = require("chalk");

// 创建一个 renderer 实例
const { createBundleRenderer } = require("vue-server-renderer");

// 依赖的文件
const publicHtml = resolve(__dirname, "../../public/index.html");
const skeletonJson = resolve(__dirname, "./skeleton.json");
const skeletonTemplate = resolve(__dirname, "./skeleton.template.html");

// https://ssr.vuejs.org/zh/api/#createrenderer
// 创建一个 BundleRenderer 实例
const renderer = createBundleRenderer(skeletonJson, {
  template: fs.readFileSync(skeletonTemplate, "utf-8")
});
renderer.renderToString({}, (err, html) => {
  // 压缩html
  html = htmlMinifier.minify(html, {
    collapseInlineTagWhitespace: true,
    minifyCSS: true
  });
  // 重写public/index.html
  fs.writeFileSync(publicHtml, html, "utf-8");
  if (err) {
    console.log(chalk.red("骨架屏生成失败！错误：" + err));
    process.exit(1);
  }
  console.log(chalk.green("骨架屏生成成功！"));
});
```

- 骨架屏组件

```js
// src/components/skeleton/install.js
import skeleton from "./skeleton.vue";
import skeletonCircle from "./basic/skeleton-circle.vue";
import skeletonSquare from "./basic/skeleton-square.vue";
import skeletonRow from "./layout/skeleton-row.vue";
import skeletonColumn from "./layout/skeleton-column.vue";
import "./skeleton.less";

function install(Vue) {
  Vue.component(skeleton.name, skeleton);
  Vue.component(skeletonRow.name, skeletonRow);
  Vue.component(skeletonColumn.name, skeletonColumn);
  Vue.component(skeletonSquare.name, skeletonSquare);
  Vue.component(skeletonCircle.name, skeletonCircle);
}

const skeletonLoading = {
  install
};
export default skeletonLoading;
```

## 实现效果

- loading 效果

  <figure class="half">
      <img src="https://github.com/dirkhe1051931999/common-demo/blob/master/vue-skeleton/screenshot/skeleton-loading.png">
      <img src="https://github.com/dirkhe1051931999/common-demo/blob/master/vue-skeleton/screenshot/after-skeleton-loading.png">
  </figure>

- dom 对比

  <figure class="half">
      <img src="https://github.com/dirkhe1051931999/common-demo/blob/master/vue-skeleton/screenshot/skeleton-dom.png">
      <img src="https://github.com/dirkhe1051931999/common-demo/blob/master/vue-skeleton/screenshot/after-skeleton-dom.png">
  </figure>

## 源码地址

- [vue-skeleton](https://github.com/dirkhe1051931999/common-demo/tree/master/vue-skeleton)

## 参考

[一种自动化生成骨架屏的方案](https://github.com/Jocs/jocs.github.io/issues/22)
