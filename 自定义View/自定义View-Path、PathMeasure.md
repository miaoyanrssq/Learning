#自定义View-Path、PathMeasure

## 一、Path常用方法表
>为了兼容性(偷懒) 本表格中去除了部分API21(即安卓版本5.0)以上才添加的方法。

![] (image/path1.png)
![] (image/path2.png)

## 二、Path详解

**如遇问题，请尝试关闭硬件加速**

本次特地开了一篇详细讲解Path，为什么要单独摘出来呢，这是因为Path在2D绘图中是一个很重要的东西。

在前面我们讲解的所有绘制都是简单图形(如 矩形 圆 圆弧等)，而对于那些复杂一点的图形则没法去绘制(如绘制一个心形 正多边形 五角星等)，而使用Path不仅能够绘制简单图形，也可以绘制这些比较复杂的图形。另外，根据路径绘制文本和剪裁画布都会用到Path。

**Path封装了由直线和曲线(二次，三次贝塞尔曲线)构成的几何路径。你能用Canvas中的drawPath来把这条路径画出来(同样支持Paint的不同绘制模式)，也可以用于剪裁画布和根据路径绘制文字。我们有时会用Path来描述一个图像的轮廓，所以也会称为轮廓线(轮廓线仅是Path的一种使用方法，两者并不等价)**

[具体事例-基本绘制] (http://www.gcssloop.com/customview/Path_Basic/)

[具体示例-贝塞尔曲线] (http://www.gcssloop.com/customview/Path_Bezier)

[具体示例-填充模式] (http://www.gcssloop.com/customview/Path_Over)

## 三、PathMeasure
![] (image/pathmeasure.png)

### 1.构造函数
**无参构造函数：**

```
PathMeasure ()
```
用这个构造函数可创建一个空的 PathMeasure，但是使用之前需要先调用 setPath 方法来与 Path 进行关联。被关联的 Path 必须是已经创建好的，如果关联之后 Path 内容进行了更改，则需要使用 setPath 方法重新关联。

**有参构造函数：**

```
PathMeasure (Path path, boolean forceClosed)
```
用这个构造函数是创建一个 PathMeasure 并关联一个 Path， 其实和创建一个空的 PathMeasure 后调用 setPath 进行关联效果是一样的，同样，被关联的 Path 也必须是已经创建好的，如果关联之后 Path 内容进行了更改，则需要使用 setPath 方法重新关联。

该方法有两个参数，第一个参数自然就是被关联的 Path 了，第二个参数是用来确保 Path 闭合，如果设置为 true， 则不论之前Path是否闭合，都会自动闭合该 Path(如果Path可以闭合的话)。

**在这里有两点需要明确:**

> * 不论 forceClosed 设置为何种状态(true 或者 false)， 都不会影响原有Path的状态，即 Path 与 PathMeasure 关联之后，之前的的 Path 不会有任何改变。  
> * forceClosed 的设置状态可能会影响测量结果，如果 Path 未闭合但在与 PathMeasure 关联的时候设置 forceClosed 为 true 时，测量结果可能会比 Path 实际长度稍长一点，获取到到是该 Path 闭合时的状态。

### 2.getSegment

```
boolean getSegment (float startD, float stopD, Path dst, boolean startWithMoveTo)
```
![] (image/pathmeasure2.png)

> * 如果 startD、stopD 的数值不在取值范围 [0, getLength] 内，或者 startD == stopD 则返回值为 false，不会改变 dst 内容。
* 如果在安卓4.4或者之前的版本，在默认开启硬件加速的情况下，更改 dst 的内容后可能绘制会出现问题，请关闭硬件加速或者给 dst 添加一个单个操作，例如: dst.rLineTo(0, 0)

startWithMoveTo 的取值：
![] (image/pathmeasure3.png)

### 3.nextContour

我们知道 Path 可以由多条曲线构成，但不论是 getLength , getgetSegment 或者是其它方法，都只会在其中第一条线段上运行，而这个 nextContour 就是用于跳转到下一条曲线到方法，如果跳转成功，则返回 true， 如果跳转失败，则返回 false。

* 1.曲线的顺序与 Path 中添加的顺序有关。
* 2.getLength 获取到到是当前一条曲线分长度，而不是整个 Path 的长度。
* 3.getLength 等方法是针对当前的曲线

## Path & SVG

我们知道，用Path可以创建出各种个样的图形，但如果图形过于复杂时，用代码写就不现实了，不仅麻烦，而且容易出错，所以在绘制复杂的图形时我们一般是将 SVG 图像转换为 Path。

你说什么是 SVG?

SVG 是一种矢量图，内部用的是 xml 格式化存储方式存储这操作和数据，你完全可以将 SVG 看作是 Path 的各项操作简化书写后的存储格式。

[具体实例-待补充] ()