# OSSMaker 使用详解

项目中,当我们要在列表页展示图片时,为了提升图片的加载速度,同时兼顾显示效果。一般我们会采用展示缩略图的形式去加载图片。这时我们就会用到阿里云OSS的图片地址拼接策略。

## Aliyun图片裁剪拼接参数简介

### 图片缩放

操作名称：`resize`

| 名称 | 描述 | 取值范围 |当前是否支持|
| --- | --- | --- |----|
| m | 指定缩略的模式：lfit：等比缩放，限制在指定w与h的矩形内的最大图片。mfit：等比缩放，延伸出指定w与h的矩形框外的最小图片。fill：固定宽高，将延伸出指定w与h的矩形框外的最小图片进行居中裁剪。pad：固定宽高，缩略填充。fixed：固定宽高，强制缩略 | lfit、mfit、fill、pad、fixed，默认为lfit。  | 已支持 |
| w |指定目标缩略图的宽度。|1-4096| 已支持|
|h|指定目标缩略图的高度。	|1-4096| 已支持|
|l|指定目标缩略图的最长边。|1-4096| 已支持|
|s|指定目标缩略图的最短边。|1-4096| 已支持|
|limit|指定当目标缩略图大于原图时是否处理。值是 1 表示不处理；值是 0 表示处理。|0/1, 默认是 1|未支持|
|color|当缩放模式选择为pad（缩略填充）时，可以选择填充的颜色(默认是白色)参数的填写方式：采用16进制颜色码表示，如00FF00（绿色）。|[000000-FFFFFF]|未支持|


`注意:`
* 1 对于原图：
> 图片格式只能是：jpg、png、bmp、gif、webp、tiff。
> 文件大小不能超过20 MB。
> 使用图片旋转时图片的宽或者高不能超过4096。

* 2 对于缩略图：对缩略后的图片大小有限制，目标缩略图宽与高的乘积不能超过 4096 x 4096，且单边长度不能超过 4096 x 4。

* 3 当只指定宽度或者高度时，在等比缩放的情况下，都会默认进行单边的缩放。在固定宽高的模式下，会默认宽高一样的情况下进行缩略。
* 4 如果只指定宽度或者高度，原图按原图格式返回。如果想保存成其他格式，详细可以查看质量变换及格式转换。
* 5 调用resize，默认是不允许放大。即如果请求的图片对原图大，那么返回的仍然是原图。如果想取到放大的图片，即增加参数调用limit,0 （如：https://image-demo.oss-cn-hangzhou.aliyuncs.com/example.jpg?x-oss-process=image/resize,w_500,limit_0）

### 图片裁剪

#### 内切圆

操作名称：`circle`

| 参数 | 描述 | 取值 |是否支持|
| --- | --- | --- |---|
| r  | 从图片取出的圆形区域的半径	 | 半径 r 不能超过原图的最小边的一半。如果超过，则圆的大小仍然是原圆的最大内切圆 | 已支持|

`注意：`

* 如果图片的最终格式是 png、webp、 bmp 等支持透明通道的图片，那么图片非圆形区域的地方将会以透明填充。如果图片的最终格式是 jpg，那么非圆形区域是以白色进行填充。推荐保存成 png 格式。

* 如果指定半径大于原图最大内切圆的半径，则圆的大小仍然是图片的最大内切圆。

### 圆角矩形

操作名称：`rounded-corners`

| 参数 | 描述 | 取值 |是否支持|
| --- | --- | --- |---|
| r  | 将图片切出圆角，指定圆角的半径。 | [1, 4096] 生成的最大圆角的半径不能超过原图的最小边的一半。 | 已支持|


## 之前的做法

对于图片的展示一般我们提前知晓图片要展示的宽高,或者知道图片的高度宽度自适应或者知道宽度高度自适应。

### 定义一个该位置图片需要拼接的字符串常量

`static NSString *URL_SCALE_DUBLIST_AVATAR = @"x-oss-process=image/resize,m_mfit,h_210,w_210/rounded-corners,r_9/format,png";
`

当我们需要展示的时候 将这个常量直接添加到我们要展示的图片的后面,就可以拿到我们要展示的缩略图了。

### 问题

上面的这种做法最直观同时也最简单。但是如果每个列表展示的图片宽高均不相同 那么我们就要定义无数个字符串常量来满足我们的需求。

## 改进方法

由于大多数情况下我们展示的图片宽高都不尽相同,这些字符串常量基本上复用的可能性不是太大(当然为了提高图片缓存的利用率我们需要跟UI协调尽量统一)。

如果我们不去定义每处展示的字符串常量,那么我们可以通过在每个地方单独通过参数的方式传给我们的管理对象 我需要设置的属性(宽度/高度/圆角/内切圆/图片展示模式)。由于这里要传递的参数比较多,因此通过普通的工厂方法去实现的话会导致代码的可读性方面降低。同时使用者也需要书写大量的代码。因此,这里 推出链式调用的方式去拼接字符串。

### 使用示例

```objc
NSString *res = self.coverURL.absoluteString.maker.resize.contentMode(@(OSSImageResizeContentModelTypeMfit)).height(@60).width(@106).resultString;

```

这里是我们想要将图片切为宽度是106 高度为60 contentMode为Fit样式的展示。

`注意:` 最终实际拼接出的图片的`宽高`为根据外部传入的参数 `乘以` 当前屏幕的 `Scale`,因此会根据当前设备去取2x还是3x图。

通过上面这样的拼接我们就可以将我们需要设置的各项参数拼接到图片URL的尾部。

## 内部实现简介

为了方便使用,这里将图片拼接的`maker`方法放到了字符串的分类中。

```objc
- (ULOSSImageMaker *)maker
{
    if (!objc_getAssociatedObject(self, &makerName)) {
        ULOSSImageMaker  *maker = [[ULOSSImageMaker alloc] initWithBaseURLString:self];
        objc_setAssociatedObject(self, &makerName, maker, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }
    return objc_getAssociatedObject(self, &makerName);
}
```

在这里我们创建了一个 `ULOSSImageMaker` 对象,在这个类中包含两个方法: 

* 初始化方法 

```objc
- (instancetype)initWithBaseURLString:(NSString *)aURLString
{
    if (self = [super init]) {
        self.baseURLString = [NSString stringWithFormat:@"%@?x-oss-process=image",aURLString];
    }
    return self;
}
```

* resize的实现方法

```objc
- (ULOSSImageResize *)resize
{
    NSString *actionString = [self.baseURLString stringByAppendingString:@"/resize"];
    ULOSSImageResize *action = [[ULOSSImageResize alloc] initWithBaseURLString:actionString];
    return action;
}
```

这里在`resize`方法中我们我们创建了一个`ULOSSImageResize`对象,几乎所有的针对图片的缩放操作我们都放在了这个对象中进行。


## 具体代码实现

设置宽度

```objc

- (ResizeHandler)width
{
    return ^(NSNumber *attr) {
        self.oss_w = [NSNumber numberWithFloat:attr.floatValue*self.curScale];
        [self addOSSImageAttributeValue:self.oss_w.stringValue type:oss_w];
        return self;
    };
}
```

链接拼接：

```objc
- (void)addOSSImageAttributeValue:(NSString *)atrri type:(NSString *)type
{
    NSString *tem = [NSString stringWithFormat:@",%@%@", type, atrri];
    self.baseURLString = [self.baseURLString stringByAppendingString:tem];
}
```

## 还存在的问题

* 对于属性的支持目前还没有全部支持 只是支持了项目中使用到的
* 对于枚举类型的支持提示效果不佳 且需要转换成NSNumber类型

## 项目中的使用

### 头像

通过对之前使用ULImageScaleTypeHeader以及ULImageScaleBigTypeHeader以及ULImageScaleTypeBigSquareHeader三类头像 以及在代码中各位置使用时的实际大小

现决定提供三种大小的头像 `30*30 60*60 90*90` 三种大小尺寸的头像

## 总结

OSS拼接本身是一个跟业务耦合度很低的逻辑,因此在实现的时候我们尽量将方法设置的尽可能的简单。通过链式调用的方式不仅可以很好的提示使用者使用方法,同时也降低了与业务代码的耦合。






