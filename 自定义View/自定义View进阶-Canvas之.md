# 安卓自定义View进阶-Canvas

## 一、Canvas简介
Canvas我们可以称之为画布，能够在上面绘制各种东西，是安卓平台2D图形绘制的基础，非常强大。

**一般来说，比较基础的东西有两大特点:**

* 1.可操作性强：由于这些是构成上层的基础，所以可操作性必然十分强大。
* 2.比较难用：各种方法太过基础，想要完美的将这些操作组合起来有一定难度。
  
不过不必担心，本系列文章不仅会介绍到Canvas的操作方法，还会简单介绍一些设计思路和技巧。
## 二、Canvas的常用操作功能表

![] (image/canvas函数表.png)

PS： Canvas常用方法在上面表格中已经全部列出了，当然还存在一些其他的方法未列出，具体可以参考官方文档 [Canvas] (http://developer.android.com/reference/android/graphics/Canvas.html)

## 三、Canvas基本操作
### 1.画布操作

为什么要有画布操作？  

画布操作可以帮助我们用更加容易理解的方式制作图形。

例如： 从坐标原点为起点，绘制一个长度为20dp，与水平线夹角为30度的线段怎么做？

按照我们通常的想法(被常年训练出来的数学思维)，就是先使用三角函数计算出线段结束点的坐标，然后调用drawLine即可。

然而这是否是被固有思维禁锢了？

假设我们先绘制一个长度为20dp的水平线，然后将这条水平线旋转30度，则最终看起来效果是相同的，而且不用进行三角函数计算，这样是否更加简单了一点呢？

**合理的使用画布操作可以帮助你用更容易理解的方式创作你想要的效果，这也是画布操作存在的原因。**

**PS: 所有的画布操作都只影响后续的绘制，对之前已经绘制过的内容没有影响。**

####（1）位移（translate）
translate是坐标系的移动，可以为图形绘制选择一个合适的坐标系。 **请注意，位移是基于当前位置移动，而不是每次基于屏幕左上角的(0,0)点移动**

```
canvas.translate(200,200);
```
#### (2)缩放（scale）
缩放提供了两个方法：

```
public void scale (float sx, float sy)

public final void scale (float sx, float sy, float px, float py)
```
这两个方法中前两个参数是相同的分别为x轴和y轴的缩放比例。而第二种方法比前一种多了两个参数，用来控制缩放中心位置的。  

缩放比例(sx,sy)取值范围详解：
![] (image/scale.png)

缩放的中心默认为坐标原点,而缩放中心轴就是坐标轴

#### (3) 旋转（ratate）

```
public void rotate (float degrees)

public final void rotate (float degrees, float px, float py)
```
和缩放一样，第二种方法多出来的两个参数依旧是控制旋转中心点的。

默认的旋转中心依旧是坐标原点

#### (4) 错切（skew）
skew这里翻译为错切，错切是特殊类型的线性变换。

错切只提供了一种方法：

```
public void skew (float sx, float sy)
```
参数含义： 
 
* float sx:将画布在x方向上倾斜相应的角度，sx倾斜角度的tan值，
* float sy:将画布在y轴方向上倾斜相应的角度，sy为倾斜角度的tan值.

变换后:

```
X = x + sx * y
Y = sy * x + y
```

**以上集中操作都是可以叠加的**

#### （5）快照（save）和回滚（restore）
Q: 为什存在快照与回滚
A：画布的操作是不可逆的，而且很多画布操作会影响后续的步骤，例如第一个例子，两个圆形都是在坐标原点绘制的，而因为坐标系的移动绘制出来的实际位置不同。所以会对画布的一些状态进行保存和回滚。

![] (image/save_restore.png)

状态栈：

![] (image/状态栈.jpg)

这个栈可以存储画布状态和图层状态。

Q：什么是画布和图层？

A：实际上我们看到的画布是由多个图层构成的，如下图(图片来自网络)
![] (image/layer.jpg)

实际上我们之前讲解的绘制操作和画布操作都是在默认图层上进行的。
在通常情况下，使用默认图层就可满足需求，但是如果需要绘制比较复杂的内容，如地图(地图可以有多个地图层叠加而成，比如：政区层，道路层，兴趣点层)等，则分图层绘制比较好一些。
你可以把这些图层看做是一层一层的玻璃板，你在每层的玻璃板上绘制内容，然后把这些玻璃板叠在一起看就是最终效果。

**save**

save 有两种方法：

```
// 保存全部状态
public int save ()

// 根据saveFlags参数保存一部分状态
public int save (int saveFlags)
```
可以看到第二种方法比第一种多了一个saveFlags参数，使用这个参数可以只保存一部分状态，更加灵活，这个saveFlags参数具体可参考表格中的内容. 

SaveFlags

![] (image/saveFlag.png)

### 2.绘制图片
#### （1）drawPicture
**使用Picture前请关闭硬件加速，以免引起不必要的问题！**

**使用Picture前请关闭硬件加速，以免引起不必要的问题！**

**使用Picture前请关闭硬件加速，以免引起不必要的问题！**

使用很少，暂不介绍

#### （2）drawBitmap

```
// 第一种
public void drawBitmap (Bitmap bitmap, Matrix matrix, Paint paint)

// 第二种
public void drawBitmap (Bitmap bitmap, float left, float top, Paint paint)

// 第三种
public void drawBitmap (Bitmap bitmap, Rect src, Rect dst, Paint paint)
public void drawBitmap (Bitmap bitmap, Rect src, RectF dst, Paint paint)
```
相关参数自查

### 3.绘制文字

```
// 第一类
public void drawText (String text, float x, float y, Paint paint)
public void drawText (String text, int start, int end, float x, float y, Paint paint)
public void drawText (CharSequence text, int start, int end, float x, float y, Paint paint)
public void drawText (char[] text, int index, int count, float x, float y, Paint paint)

// 第二类
public void drawPosText (String text, float[] pos, Paint paint)
public void drawPosText (char[] text, int index, int count, float[] pos, Paint paint)

// 第三类
public void drawTextOnPath (String text, Path path, float hOffset, float vOffset, Paint paint)
public void drawTextOnPath (char[] text, int index, int count, Path path, float hOffset, float vOffset, Paint paint)
```

绘制文字部分大致可以分为三类：

* 第一类只能指定文本基线位置(基线x默认在字符串左侧，基线y默认在字符串下方)。
* 第二类可以分别指定每个文字的位置。(必须每个文字都要制定，否则crash)
* 第三类是指定一个路径，根据路径绘制文字。

