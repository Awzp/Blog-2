title: 记一次前端性能优化实战
date: 2016-3-3
categories: 前端
tags: [gulp,性能优化]
---
## 前言
这学期开学时去找了信息化中心的老师，说明了自己想往Node和Python方向走的想法，退出了信息化中心的学生工作室。  
然而上周六老师又找到了我，他们对一个电商网站项目的前端性能很不满意，希望我能给他们做一套性能优化方案。
<!-- more -->

## 需求
和老师面谈后，确定了需要下面几个主要需求：  
* 合并css/js文件减少http请求
* 压缩css/js文件减少流量
* 原来的模板中每个小icon都是一张图片，用雪碧图/base64减少http请求
* 大图懒加载，减轻后端压力

## 方案
常见的构建方案就是grunt,gulp,webpack,make,npm script，还有国内百度的fis。  
经过了筛选省下了gulp和fis3两个选择，不得不说fis3已经做好了一套默认方案，几乎和信息中心的要求相符，然而经过测试发现这个项目使用gulp构建更加快一些。  
确定了gulp后，接下来就是对照需求选插件了。
* 开发时实时监听变化自动刷新
借助`gulp-livereload`插件和chrome的`livereload`插件，`gulpfile`中创建名为`watch`的`task`，监听文件变动浏览器自动刷新，解放浏览器的刷新按钮和键盘的F5。
* 合并js/css文件
为了方便开发时多个文件引入，和模块化的开发，一开始使用了`gulp-usemin`插件，使用起来确实挺方便的，然而发现在多个页面构建时就废了，查了一下发现npm建议使用`gulp-useref`代替。  
然而接着发现`gulp-useref`不能将外联转化为内联，而且在流中处理了文件名后不会像`gulp-usemin`自动修改引用文件的地址，好在有`gulp-rev-replace`插件可以自动在流中修改html文件中的引用地址。  
而将外联文件合并为内联文件则可以用`gulp-inline-source`插件，和`gulp-usemin`差不多。
* js/css文件压缩
分别用`gulp-uglify`和`gulp-cssnano`插件，这个没什么好说的。
* 版本更迭css/js文件冗余
常见的有文件名加时间戳和hash两种方案，选哪个倒感觉无所谓了，这边用`gulp-rev`插件实现hash冗余文件，服务器端静态资源半年清理个一次也就差不多了。
* 小图优化
常见的就是把小图转成字体文件，SVG，雪碧图，base64这几种方案。  
这边选择了base64，其实base64有利有害，合并进css文件会变大1/3左右，而且css文件太大后会延长css文件渲染时间。这边我设定的5KB为阈值，css文件大概增加了100k，影响不大，图片跟着css文件一起缓存，倒也不错。
* 大图懒加载
由于原来就使用了jquery，于是用了百度改过的jquery图片懒加载插件，这里遇到了一个大坑。  
假设`html`中有两个`div`，通过css将第一块放在左边当作侧栏，第二块放在右边当作主栏，将所有图片都做懒加载处理，如果左栏高度过高，左栏看完了右栏才能开始加载。  
鹅厂的大牛张鑫旭曾经也写过一个这样的插件。这类的插件原理都类似，大致看了下源码，猜测是插件设计时没有想到这种场景，所以出现了这种蜜汁bug。  
突然想起来，百度说他的百度地图和糯米都用了这个插件，那不如去看看他们怎么解决的。  
看了下糯米的页面，原来他们不对侧栏做懒加载处理。我取消了侧栏的懒加载处理，果然就正常了。

## 未完待续
这是第一次将这些优化方案在这样一个较大的项目中使用，这个项目还没有结束，如果有新的变化视情况来更新。
