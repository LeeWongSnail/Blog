相信目前大部分APP都已经支持了WebP格式的图片,下面我们通过这种图片简单介绍下WebP格式图片的优点。

### 简介

WebP 的优势体现在它具有更优的图像数据压缩算法，能带来更小的图片体积，而且拥有肉眼识别无差异的图像质量；同时具备了无损和有损的压缩模式、Alpha 透明以及动画的特性，在 JPEG 和 PNG 上的转化效果都相当优秀、稳定和统一。

下面我们看下官网介绍的对于JPEG格式的图片和Gif图片压缩对比！

![](https://tva1.sinaimg.cn/large/006tNbRwly1gbc6ha4mv7j31p80dkgwp.jpg)

![](https://tva1.sinaimg.cn/large/006tNbRwly1gbc6hzbo21j30nw0oydkv.jpg)

从上面的图中我们看到webP的压缩效果还是很明显的！

### 现有方案

熟悉iOS的开发者都知道图片下载和展示的主要框架有`SDWebImage`和`YYWebImage`。当然这两个库也都支持了WebP的图片展示,下面我们先介绍下这两个现有的方案。

#### SDWebImage

在github的简介上我们看到
![](https://tva1.sinaimg.cn/large/006tNbRwly1gbc6mz3lpxj31em0fqdkj.jpg)

SD实际上支持多种图片格式的扩展!

如果项目中我们使用了SD那么我们想增加对WebP格式的图片支持,


