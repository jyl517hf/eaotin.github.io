---
layout: post
title: 自定义View
date: 2017-12-01 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: custom-view.jpg # Add image post (optional)
tags: [Android, View, Java]
---

# 自定义View

对自定义View一直很模糊，刚好最近不太忙，系统性的总结一下，自定义View主要有以下几个知识点：

- Canvas：可以它为一个画布，用于承载View中的所有元素（基础）；
- Paint：作为画笔的存在，用来加特效的（基础）；
- 裁切和几何变换：通过限定在某些区域绘制以及对view进行变换达到指定效果（进阶）；

## Canvas

可以通过drawXxx()方法画出下列简单图形

- drawCircle():圆
- drawPoint():点
- drawLine():线
- drawLines():多条线
- drawRect():长方形
- drawArc():弧
- drawOwal():椭圆
- drawRoundRect():圆角矩形
- 通过drawPath()画出复杂的图形
- 通过drawBitmap()画图片
- 通过drawText()画图片

## Paint
- setColor():设置颜色

- setShader(Shader shader):设置着色器，Shader有以下几种子类：

- LinearGradient 线性渐变，两个点之间的颜色线性渐变

> Shader.TileMode，着色规则有三种，CLAMP、MIRROR、REPEAT，以下同。

- RadialGradient 辐射渐变，从圆心向四周进行辐射渐变

- SweepGradient 扫描渐变，从圆的0角度，旋转进行扫描渐变

- BitmapShader 图片着色器，将图片作为着色器，比如可以用来做圆型头像的裁剪

- ComposeShader 混合着色器，一个Paint中使用两个着色器，混合用

>PorterDuff.Mode，是用来指定两个着色器公共绘制时的颜色策略，是一个enum，共17个。
大体分为两类，Alpha 合成 (Alpha Compositing)、混合 (Blending)

- setColorFilter(ColorFilter colorFilter):为绘制设置颜色的过滤。可用于制作滤镜效果。ColorFilter有三个子类：

- LightingColorFilter，用来模拟简单的光照效果。
- PorterDuffColorFilter，使用一个指定的颜色和一种指定的 PorterDuff.Mode 来与绘制对象进行合成。
- ColorMatrixColorFilter，使用一个 ColorMatrix 来对颜色进行处理，是一个4*5的矩阵（有点厉害，不知道怎么用），[StyleImageView](https://github.com/chengdazhi/StyleImageView)

- setXfermode(Xfermode xfermode)，只有一个子类PorterDuffXfermode
- setAntiAlias (boolean aa) 设置抗锯齿，默认为false,需要显示打开
- setStyle(Paint.Style style)，
- setStrokeWidth(float width)
- setStrokeCap(Paint.Cap cap)
- setStrokeJoin(Paint.Join join)
- setStrokeMiter(float miter)
- setDither(boolean dither)
- setFilterBitmap(boolean filter)设置是否使用双线性过滤来绘制 Bitmap 。
- setPathEffect(PathEffect effect)：PathEffect 分为两类，单一效果的  CornerPathEffect DiscretePathEffect DashPathEffect PathDashPathEffect ，和组合效果的  SumPathEffect ComposePathEffect。
- setShadowLayer(float radius, float dx, float dy, int shadowColor)
- setMaskFilter(MaskFilter maskfilter)
- setTextSize()

## 范围裁切和自由变换

- 范围裁切

>使用canvas的clipRect()和clipPath()，这两个方法进行裁切，注意使用Canvas.save()和Canvas.restore()保存和恢复Canvas的状态

- 自由变换
- 使用 Canvas 来做常见的二维变换;
> Canvas.translate(float dx, float dy) 平移  
> Canvas.rotate(float degrees, float px, float py) 旋转  
> Canvas.scale(float sx, float sy, float px, float py) 放缩  
> Canvas.skew(float sx, float sy) 错切  

- 使用 Matrix 来做常见和不常见的二维变换；

> 1.创建 Matrix 对象；  
> 2.调用 Matrix 的 pre/postTranslate/Rotate/Scale/Skew() 方法来设置几何变换;  
> 3.使用 Canvas.setMatrix(matrix) 或 Canvas.concat(matrix) 来把几何变换应用到 Canvas。  
> 4.Matrix.setPolyToPoly(float[] src, int srcIndex, float[] dst, int dstIndex, int pointCount) 用点对点映射的方式设置变换

- 使用 Camera 来做三维变换。    Camera 的三维变换有三类：旋转、平移、移动相机。注意使用Camera的sava()、restore()、reset()等方法，大致与Canvas的类似
>    1.旋转：Camera.rotateX(),Camera.rotateY(),Camera.rotateZ(),Camera.rotate() 三维旋转；注意Camera的旋转中心，一般的需要配合Canvas的translate(),将旋转中心切换为远点进行旋转；然后用camera.applyToCanvas(canvas); // 把旋转投影到 Canvas
>    2.平移：Camera.translate(float x, float y, float z)
>    3.移动相机：Camera.setLocation(x, y, z) 设置虚拟相机的位置，注意这个方法它的参数的单位不是像素，而是 inch，英寸

## 绘制顺序

- 自定义View的绘制顺序

1. 绘制背景：不能够被重写，只能通过xml中设置背景或代码中显示设置背景；
2. onDraw()：可以被重写，用于绘制主题，注意在super.onDraw()之前、之后进行绘制的区别；
3. dispatchDraw()：可以被重写，用于绘制子View,同样的注意在super.dispatchDraw()之前、之后进行绘制的区别；
4. onDrawForeground():可以被重谢，用于绘制滑动边缘渐变、滑动条、前景，注意在super.onDrawForeground()之前、之后进行绘制的区别；这个方法是在API23之后才支持。

- View的调度

draw()方法进行View绘制的调度，可以在super.draw()之前绘制，会在绘制背景之前执行，在super.draw()方法之后绘制，会在onDrawForeground()之后绘制。

- 注意点

1. **出于效率考虑，ViewGroup 默认会绕过 draw() 方法，直接执行 dispatchDraw()，以此来简化绘制流程。若此时在其他方法中绘制了View,这时候需要是用View.setWillNotDraw(false)来恢复正常绘制流程。**

2. **在重写的方法有多个选择时，优先选择 onDraw()。Android对onDraw()方法进行过优化，可以在不需要重绘时，跳过onDraw()的重复执行，提升性能**

