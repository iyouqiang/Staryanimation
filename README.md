# Staryanimation

#iOS 动画专题

##Quartz2D & CALayer & CAShapeLayer 关系 及 mask属性

###1、Quartz2D

提起iOS中的绘图控件，必然会想到Quartz2D。Quartz 2D是⼀个二维绘图引擎,同时支持iOS和Mac系统。Quartz2D的API来自于CoreGraphics框架，数据类型和函数基本都以CG作为前缀，如CGContextRef、CGPathRef。
Quartz 2D能完成的工作：

* 绘制图形 : 线条\三角形\矩形\圆\弧等
* 绘制文字
* 绘制\生成图片(图像)
* 读取\生成PDF
* 截图\裁剪图片
* 自定义UI控件

一般在开发中，给图片添加水印，合并图片，自定义UI控件（UIKit框架中没有的）等。
一般情况下，通过苹果官方的UIKit库就可以搭建常见的UI界面了，但是如果通过UIKit框架不能实现的一些UI界面，就需要我们自己来通过Quartz2D进行绘制了。
自定义UI控件是Quartz2D中非常重要的一个功能

**使用Quartz2D 绘图有2种方式：**

* 方式一：直接调用Quartz2D的API 进行绘图
* 方式二：调用 UIKit 框架封装好的 API 进行绘图（代码简单，但是只能是文字和图片）

```
1.开启图形上下文
(如果是自定义view，不需要这一步，但是必须在drawRect方法中获取context，在这里才能获取和view相关联的context)
UIGraphicsBeginImageContext(size)

2、获取图形上下文
CGContextRef context = UIGraphicsGetCurrentContext();

3.设置上下文的一些属性（颜色，线宽）
CGContextSetRGBFillColor(context, 1.0, 1.0, 1.0, 1.0);
CGContextSetLineWidth(context, 2.0);   
CGContextSetLineJoin(context, kCGLineJoin

4.拼接路径（绘制图形）
CGContextMoveToPoint(context, 50, 50);
CGContextAddLineToPoint(context, 100, 100);

5.渲染
CGContextStrokePath(context);
CGContextStrokePath(context);
CGContextFillPath(context);

6.关闭上下文(关闭路径)
CGContextClosePath(context)

方式二：

1、获取上下文
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    
2、设置属性
    CGContextSetRGBStrokeColor(ctx,180.0/255, 20.0/255, 210.0/255, 1.0);
    
3、 路径拼接
    CGMutablePathRef path = CGPathCreateMutable();
    CGPathMoveToPoint(path, NULL, 50, 50);
    CGPathAddLineToPoint(path, NULL, 100, 100);
    
4、 路径拼接到上下文中
    CGContextAddPath(ctx, path);
    
5、渲染
    CGContextStrokePath(ctx);

```

注：1.绘制的方法必须写在drawRect:方法中，只有在这里才能获取和view相关联的上下文
2.drawRect:什么时候调用（不能自己调用，只能通过系统调用这个方法）。
当view第一次显示到屏幕上时(被加到UIWindow上显示出来)
调用view的setNeedsDisplay或者setNeedsDisplayInRect:时

[iOS - Quartz2D & CALayer & CAShapeLayer](https://www.jianshu.com/p/91b0cb25d4cb)

[CGContext用法详解](https://blog.csdn.net/lvxiangan/article/details/50561977)

[IOS用CGContextRef画各种图形](https://blog.csdn.net/rhljiayou/article/details/9919713)

[Quartz2D](https://blog.csdn.net/qq_41431406/article/details/109047126)

###2、CALayer

* 在iOS中CALayer的设计主要是了为了内容展示和动画操作，CALayer本身并不包含在UIKit中，它不能响应事件。

* 由于CALayer在设计之初就考虑它的动画操作功能，CALayer很多属性在修改时都能形成动画效果，这种属性称为“隐式动画属性”。

###自定义layer

layer和view是紧密相连的，所以他们的操作很相似。自定义layer需要重写drawInContext方法，自定义view需要重写drawRect方法。针对view的操作最终都会作用到layer上

```
- (void)drawInContext:(CGContextRef)ctx {
    // 设置为蓝色
    CGContextSetRGBFillColor(ctx, 0, 0, 1, 1);
    
    //绘制图形的方法和view是一样的都是使用Quartz2D，参考Quartz2D部分

    // 渲染
    CGContextFillPath(ctx);
}

注意：需要刷新控件时，不能自己手动调用这个方法，而是使用[self setNeedsDisplay]
```

### CALayer的mask属性
mask实际上是layer内容的一个遮罩，如果我们把mask设置成透明的，实际看到的layer是完全透明的(这个layer就看不到了)，也就是说只有mask的内容不透明的部分和layer叠加的部分才会显示出来

###3、CAShapLayer

* CAShapeLayer继承自CALayer，可使用CALayer的所有属性
* CAShapeLayer需要和贝塞尔曲线配合使用才有意义。
* 使用CAShapeLayer与贝塞尔曲线可以实现不在view的DrawRect方法中画出一些想要的图形

##一、核心动画

* 演员----->CALayer,规定电影的主角是谁
* 剧本----->CAAnimation,规定电影该怎么演,怎么走,怎么变换,
* 开拍----->AddAnimation,开始执行

**1.CALayer是什么?**

CALayer是一个与UiView很类似的概念,同样有Layer,subLayer...,同样backgroundColor,frame等相似的属性,我们可以将UIView看做一个特殊的CALayer,只不过UIView可以响应事件而已,一般来说layer有两种用途,二者不相互冲突:一对view相关属性的设置包括圆角,阴影,边框等参数.二是实现对view的动画操控.因此对一个view做动画实际上是对layer进行操控.

**2.CAAnimation是什么?**

![](https://upload-images.jianshu.io/upload_images/3038868-47c00896a26530e2.png?imageMogr2/auto-orient/strip|imageView2/2/w/1000)

CAAnimation可以分为四种:

* 1.CABasicAnimation

   通过设定起始点,终点,时间,动画会沿着设定的点进行移动,可以看做特殊的CAKeyAnimation
   [iOS核心动画详解](https://www.jianshu.com/p/123568e0d73a)
   [iOS核心动画系列](https://www.cnblogs.com/lizhishuai/p/5817310.html)

* 2.CAKeyFrameAnimation

     keyAnimation顾名思义是关键点就是frame,你可以通过设定layer的起始点,中间点,终点的frame时间,动画就会沿着你设定的轨迹进行移动
     
    CABasicAnimation与CAKeyframeAnimation区别：
     
    CAKeyframeAnimation可以说是CABasicAnimation的一个升级版。
    CABasicAnimation只能从一个值变化到另一个值，而CAKeyframeAnimation提供了一个数组，里面提供可以提供一系列的关键值。
    
    [iOS CAKeyframeAnimation关键帧动画](https://blog.csdn.net/guoyongming925/article/details/112212708)

* 3.CAAnimationGroup

   group就是组的意思,layer设定了很多动画,吧他们组合起来同时执行

* 4.CATransition

   这个就是苹果帮开发者封装好的一些动画
   
     [iOS动画之-CATransition转场动画](https://www.jianshu.com/p/51d8a375d554)


##二、渐变动画
CAGradientLayer是CALayer的另一个子类，专门用于渐变的图层。

##三、贝塞尔曲线
什么是贝塞尔曲线：简单的说贝塞尔曲线通过控制曲线上的起始点、终止点以及中间的控制点来生成曲线的路径。贝塞尔曲线又分为一阶曲线、二阶曲线、三阶曲线和高阶曲线。

* 一阶曲线

![](https://upload-images.jianshu.io/upload_images/5305626-61d182ae1183294a.image?imageMogr2/auto-orient/strip|imageView2/2/w/360)

* 二阶曲线

![](https://upload-images.jianshu.io/upload_images/5305626-2667136bfff29be4.image?imageMogr2/auto-orient/strip|imageView2/2/w/360)

* 三阶曲线

![](https://upload-images.jianshu.io/upload_images/5305626-f0d9c9430c74964a.image?imageMogr2/auto-orient/strip|imageView2/2/w/360)

* 四阶曲线

![](https://upload-images.jianshu.io/upload_images/5305626-fbf57c317e085e8e.image?imageMogr2/auto-orient/strip|imageView2/2/w/360)

贝塞尔曲线在iOS中有一个对应的类，叫做UIBezierPath，UIBezierPath是Core Graphics框架关于路径的封装，我们可以根据它，画出直线、椭圆、圆、曲线等各种形状。
UIBezierPath通常与CAShapeLayer配合使用。CAShapeLayer继承自CALayer，它拥有CALayer的所有属性。UIBezierPath为CAShapeLayer提供路径， CAShapeLayer在提供的路径中进行渲染， 绘制出图形。

[iOS中的贝塞尔曲线(UIBezierPath)](https://www.jianshu.com/p/4b4a99a5e6eb)

##四、粒子动画

粒子动画主要是通过**CAEmitterLayer**和**CAEmitterCell**来完成。使用粒子动画可以实现很多的特效，比如红包雨，点击、星星等。

###CAEmitterLayer

CAEmitterLayer是CALayer的子类，该类用于实现粒子发射器。通过它来发射粒子来展现动画。CAEmitterLayer常用属性如下所示：

###CAEmitterCell

**CAEmitterCell**是一个粒子单元，是通过CAEmitterLayer进行发射的一个粒子单元，CAEmitterLayer也可以发射粒子。

[iOS动画之-CAEmitterLayer粒子动画](https://www.jianshu.com/p/6d55c3e1ed71)

##五、仿真动画

UIDynamicAnimator是ios7新加的一个仿真动画，用于模拟现实世界中的物理模型，用于实现现实生活中的一些物理现象。

###UIDynamicAnimator
UIDynamicAnimator是一个运动管理者，用于指定实现动画的视图和添加动画。

###UIDynamicBehavior
UIDynamicBehavior是一个用于创建运动行为的基类,派生的子类主要包括以下几个：

* UIAttachmentBehavior(吸附行为),
* UICollisionBehavior(碰撞行为),
* UIDynamicItemBehavior(动力元素行为),
* UIGravityBehavior(重力行为)
* UIPushBehavior(推动行为)
* UISnapBehavior(吸附行为)

https://www.jianshu.com/p/7702ba5aa0e9

##六、json动画

###Lottie
Lottie动画是Airbnb开源的一个支持 Android、iOS 以及 ReactNative。通过AE导出的JSON文件+Lottie库可快速实现动画绘制。

[Lottie动画原理](https://cloud.tencent.com/developer/article/1655236)

##七、SVGA动画

SVGA 是一种跨平台的开源动画格式，同时兼容iOS / Android / Web。SVGA 除了使用简单，性能卓越，同时让动画开发分工明确，各自专注各自的领域，大大减少动画交互的沟通成本，提升开发效率。动画设计师专注动画设计，通过工具输出 svga 动画文件，提供给开发工程师在集成 svga player 之后直接使用。

[git-SVGAPlayer-iOS](https://github.com/svga/SVGAPlayer-iOS)

[动画库 - SVGA-iOS - 对比](https://www.jianshu.com/p/469b94c04ffd)


