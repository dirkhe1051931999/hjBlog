# 你必须要经历的 web 页面性能优化之旅

![](./screenshot/00.png)

由浅至深的总结 web 页面性能优化

## 优化前置知识

1. [chrome 开发者工具-Network](https://github.com/dirkhe1051931999/hjBlog/blob/master/blog-web-optimize/lessons/01.md)
2. chromechrome 开发者工具之 Performance、Memory
3. url 的解析过程
4. nginx 的基础知识
5. 哪些指标能够衡量页面性能
6. 解耦、颗粒度、吞吐量
7. PWA--降低首屏渲染时间，极大提升体验
8. 图片优化-- webp 格式
9. http 优化--三次握手/四次挥手

## html/css/js 中的优化

1. 防抖、节流、懒加载、预加载
2. cdn、雪碧图、合并资源、缓存/缓存 dom、async/defer/异步加载

## http 的优化

1. 负载均衡
2. gz

## vue 中的优化

1. Vue 应用运行时性能优化措施

   引入生产环境的 Vue 文件

   使用单文件组件预编译模板

   提取组件的 CSS 到单独到文件

   利用 Object.freeze()提升性能

   扁平化 Store 数据结构

   合理使用持久化 Store 数据

   组件懒加载

2. Vue 应用加载性能优化措施

   服务端渲染 / 预渲染

   组件懒加载

## webpack 的优化

## 其他