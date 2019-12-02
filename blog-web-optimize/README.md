# 你必须要经历的 web 页面性能优化之旅

<img src="./screenshot/00.jpg" width="100%">

由浅至深的总结 web 页面性能优化

## 优化前置知识

1. [chrome 开发者工具-Network](https://github.com/dirkhe1051931999/hjBlog/blob/master/blog-web-optimize/lessons/01.md)
2. [chrome 开发者工具-Performance(1)](https://juejin.im/post/5c009115f265da612859d8e2)
3. [chrome 开发者工具-Performance(2)](https://zhuanlan.zhihu.com/p/29879682)
4. [chrome 开发者工具-Performance 的 API](https://github.com/dirkhe1051931999/hjBlog/blob/master/blog-web-optimize/lessons/02.md)
5. [浏览器输入 URL 发生了什么](https://github.com/dirkhe1051931999/hjBlog/blob/master/blog-web-optimize/lessons/03.md)
6. [nginx 的反向代理与负载均衡](https://github.com/dirkhe1051931999/hjBlog/blob/master/blog-web-optimize/lessons/04.md)
7. [哪些指标能够衡量页面性能](https://github.com/dirkhe1051931999/hjBlog/blob/master/blog-web-optimize/lessons/05.md)
8. [简单理解解耦、颗粒度、吞吐量](https://github.com/dirkhe1051931999/hjBlog/blob/master/blog-web-optimize/lessons/06.md)
9. [PWA--降低首屏渲染时间，极大提升体验](https://github.com/dirkhe1051931999/hjBlog/blob/master/blog-web-optimize/lessons/07.md)
10. [图片优化-- webp 格式](https://github.com/dirkhe1051931999/hjBlog/blob/master/blog-web-optimize/lessons/08.md)
11. [http 优化--三次握手/四次挥手](https://juejin.im/post/5b83b0bfe51d4538c63131a8#heading-17)

## vue 页面性能优化实践

- 前端

1. 图片压缩，图标尽量使用 webfont，非得用图片，尽量 webpack 打包成 base64，如果不考虑兼容性，考虑一下.webp，页面有大量图片（非运营活动）使用图片懒加载
2. vue 代码的优化

   - vue-router 的路由懒加载（不用兼容到安卓 4.3）
   - 组件懒加载（不用兼容到安卓 4.3）
   - 不需要派发更新的数据，不要依赖收集，具体做法：在 created 生命周期中使用 this.xx 定义变量
   - 及时清除不必要的定时器，避免递归，闭包的使用
   - 骨架屏的注入，让用户知道页面有内容，其实是一段白屏时间
   - 针对业务需求，看能否使用 ssr
   - 输入框的[防抖与节流](https://github.com/dirkhe1051931999/hjBlog/blob/master/blog-web-optimize/lessons/09.md)

3. [webpack 优化](https://github.com/dirkhe1051931999/common-demo/blob/master/webpack-study-notes)

   - 压缩 css，合并 css，css 预加载 preload，异步加载 js
   - 开启 gzip 是有效减少代码体积的方式
   - 使用 noParse 忽略不会依赖的文件，比如项目引入一个 jquery（尽量避免使用 zepto 和 jquery）
   - 使用 exclude 和 include 做限制配置优化
   - [tree shaking](http://www.fly63.com/article/detial/4027)（webpack4 自带优化）
   - [scoping hosting](http://www.fly63.com/article/detial/4027)，优化函数声明，减少函数作用域（webpack4 自带优化，前提是使用 import 导入依赖）
   - 使用 webpack analyse，分析各包体积，有针对性优化从而减少依赖库的体积

- 服务端

1. TTFB 请求时间优化
2. 负载均衡
3. 缓存策略
4. http 优化，使用 http/2，http/2 会使用 HEAD 压缩
5. keep-alive 长链接

- 网络请求

1. 使用 cdn 合并第三方 js 情况，减少 http 请求数
2. 优化 DNS 解析时间
3. 开启 gzip（如果服务端开启，webpack 就不用 gzip 了）
4. nginx 开启缓存（缓存有利有弊，我碰到一个 js 文件缓存是 30 年，修改线上的图片，需要更改图片的名字。。。）
