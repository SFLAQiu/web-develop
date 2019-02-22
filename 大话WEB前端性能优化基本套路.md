# 大话WEB前端性能优化基本套路

## 前言   
![image](http://blog.thankbabe.com/imgs/xnyh.jpg)   
前端性能优化这是一个老生常谈的话题，但是还是有很多人没有真正的重视起来，或者说还没有产生这种意识。

当用户打开页面，首屏加载速度越慢，流失用户的概率就越大，在体验产品的时候性能和交互对用户的影响是最直接的，推广拉新是一门艺术，用户的留存是一门技术，拉进来留住用户，产品体验很关键，这里我以[美柚](https://www.meiyou.com/)的页面为例子，用实例展开说明前端优化的基本套路（适合新手上车）。




---


## WEB性能优化套路

### 基础套路1：减少资源体积
* css
    * 压缩 
    * 响应头GZIP    
    ![image](http://blog.thankbabe.com/imgs/cssys.jpg)

* js
    * 压缩 
    * 响应头GZIP   
    ![image](http://blog.thankbabe.com/imgs/jsys.jpg)

* html
    * 输出压缩
    * 响应头GZIP   
    ![image](http://blog.thankbabe.com/imgs/htmlys.jpg)

* 图片
    * 压缩
    * 使用Webp格式   
  
    ![image](http://blog.thankbabe.com/imgs/znt-webp.jpg)   
* cookie
    * 注意cookie体积，合理设置过期时间

---

### 基础套路2：控制请求数

* js
    * 合并
* css
    * 合并 
* 图片
    * 合并    
    ![image](http://blog.thankbabe.com/imgs/tbys.jpg)   
    * base64(常用图标：如logo等)
    ![image](http://blog.thankbabe.com/imgs/basetp.jpg)    
* 接口
    * 数量控制 
    * 异步ajax
* 合理使用缓存机制
    * 浏览器缓存
* js编码
    * 异步加载js
        * Require.JS 按需加载 
    * lazyload图片

---

### 基础套路3：静态资源CDN

* 请求走CDN
    * html
    * image
    * js
    * css

---

### 综合套路  

* 图片地址独立域名
    * 与业务不同域名可以减少请求头里不必要的cookie传输    
* 提高渲染速度
    * js放到页面底部，body标签底部
    * css放到页面顶部，head标签里
* 代码
    * 代码优化：css/js/html
    * 预加载，如：分页预加载，快滚动到底部的时候以前加载下一页数据
 
 ---
 
<!--### 高级套路

* 前后端分离
    * 中间层
        * nodejs服务做分离中间层
        * php服务做中间层
    * 优势
        * 职责更佳清晰，后端只需关注数据层，业务数据逻辑处理
        * 前端可以对数据做逻辑处理，api聚合
        * SEO友好，体验友好，视图模板引擎直接进行数据绑定，可以无需异步加载，浏览器直接渲染

------>

### 拓展资料
* [移动H5前端性能优化指南](https://isux.tencent.com/h5-performance.html)
* [Web性能优化：图片优化](http://www.cnblogs.com/wizcabbit/p/web-image-optimization.html)
* [WebP 探寻之路](http://isux.tencent.com/introduction-of-webp.html)
* [浅谈浏览器http的缓存机制](http://www.cnblogs.com/vajoy/p/5341664.html)
* [常见的前端性能优化手段都有哪些？都有多大收益？](https://www.zhihu.com/question/40505685)
* [前端性能优化相关](https://github.com/wy-ei/notebook/issues/34)

### 性能辅助工具
* [智图-Webp](http://zhitu.isux.us/)
* [谷歌 PageSpeed Insights](https://developers.google.com/speed/pagespeed/)(网页载入速度检测工具，需要翻墙)
* [入门Webpack，看这篇就够了](http://www.jianshu.com/p/42e11515c10f)
* [前端构建工具gulpjs的使用介绍及技巧](http://www.cnblogs.com/2050/p/4198792.html)
* [Gulp 入门指南](https://github.com/nimojs/gulp-book)

---

看完上面的套路介绍
* 可能有人会说：我在前端界混了这么多年，这些我都知道，只不过我不想去做
    * 我答： 知道做不到，等于不知道
* 也可能有人会说：压缩合并等这些操作好繁琐，因为懒，所以不做
    * 我答： 现在前端构建工具都很强大，如:grunt、gulp、webpack，支持各种插件操作，还不知道就说明你OUT了

---

因为我主要负责后端相关工作，前端并不是我擅长的，但是平时也喜欢关注前端前沿技术，这里以我的视角和开发经验梳理出基本套路，
套路点到为止，具体实施可以通过拓展资料进行深入了解，如有疑义或者补充请留言怼。

---

> 发布时间：2017-07-05