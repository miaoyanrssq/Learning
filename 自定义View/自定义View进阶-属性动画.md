# 自定义View进阶-属性动画


##ViewPropertyAnimator

具体可以跟的方法以及方法所对应的 View 中的实际操作的方法如下图所示：
![] (image/animator.png)

从图中可以看到， View 的每个方法都对应了 ViewPropertyAnimator 的两个方法，其中一个是带有 -By 后缀的，例如，View.setTranslationX() 对应了 ViewPropertyAnimator.translationX() 和 ViewPropertyAnimator.translationXBy() 这两个方法。其中带有 -By() 后缀的是增量版本的方法，例如，translationX(100) 表示用动画把 View 的 translationX 值渐变为 100，而 translationXBy(100) 则表示用动画把 View 的 translationX 值渐变地增加 100。

##ObjectAnimator

使用方式：

1.如果是自定义控件，需要添加 setter / getter 方法；  
2.用 ObjectAnimator.ofXXX() 创建 ObjectAnimator 对象；  
3.用 start() 方法执行动画。

```
public class SportsView extends View {  
     float progress = 0;

    ......

    // 创建 getter 方法
    public float getProgress() {
        return progress;
    }

    // 创建 setter 方法
    public void setProgress(float progress) {
        this.progress = progress;
        invalidate();
    }

    @Override
    public void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        ......

        canvas.drawArc(arcRectF, 135, progress * 2.7f, false, paint);

        ......
    }
}

......

// 创建 ObjectAnimator 对象
ObjectAnimator animator = ObjectAnimator.ofFloat(view, "progress", 0, 65);  
// 执行动画
animator.start();  
```

### 1.setDuration(int duration)设置动画时长（单位：毫秒）

```
// imageView1: 500 毫秒
imageView1.animate()  
        .translationX(500)
        .setDuration(500);

// imageView2: 2 秒
ObjectAnimator animator = ObjectAnimator.ofFloat(  
        imageView2, "translationX", 500);
animator.setDuration(2000);  
animator.start();  
```

###2.setInterpolator(Interpolator interpolator) 设置Interpolator

Interpolator 其实就是速度设置器。你在参数里填入不同的 Interpolator ，动画就会以不同的速度模型来执行。

```
// imageView1: 线性 Interpolator，匀速
imageView1.animate()  
        .translationX(500)
        .setInterpolator(new LinearInterpolator());

// imageView: 带施法前摇和回弹的 Interpolator
ObjectAnimator animator = ObjectAnimator.ofFloat(  
        imageView2, "translationX", 500);
animator.setInterpolator(new AnticipateOvershootInterpolator());  
animator.start();
```

简单介绍一下每个Interpolator：

* **AccelerateDecelerateInterpolator**. 
先加速再减速。这是默认的 Interpolator，也就是说如果你不设置的话，那么动画将会使用这个 Interpolator。

* **LinearInterpolator**  
  匀速
  
* **AccelerateInterpolator**  
持续加速。

* **DecelerateInterpolator**  
持续减速直到 0。

* **AnticipateInterpolator**  
先回拉一下再进行正常动画轨迹。效果看起来有点像投掷物体或跳跃等动作前的蓄力。

* **OvershootInterpolator**  
动画会超过目标值一些，然后再弹回来。效果看起来有点像你一屁股坐在沙发上后又被弹起来一点的感觉。

* **AnticipateOvershootInterpolator**  
上面这两个的结合版：开始前回拉，最后超过一些然后回弹。

* **BounceInterpolator**  
在目标值处弹跳。有点像玻璃球掉在地板上的效果。

* **CycleInterpolator**  
这个也是一个正弦 / 余弦曲线，不过它和 AccelerateDecelerateInterpolator 的区别是，它可以自定义曲线的周期，所以动画可以不到终点就结束，也可以到达终点后回弹，回弹的次数由曲线的周期决定，曲线的周期由 CycleInterpolator() 构造方法的参数决定。

* **PathInterpolator**

自定义动画完成度 / 时间完成度曲线。

用这个 Interpolator 你可以定制出任何你想要的速度模型。定制的方式是使用一个 Path 对象来绘制出你要的动画完成度 / 时间完成度曲线。例如：

```
Path interpolatorPath = new Path();

...

// 先以「动画完成度 : 时间完成度 = 1 : 1」的速度匀速运行 25%
interpolatorPath.lineTo(0.25f, 0.25f);  
// 然后瞬间跳跃到 150% 的动画完成度
interpolatorPath.moveTo(0.25f, 1.5f);  
// 再匀速倒车，返回到目标点
interpolatorPath.lineTo(1, 1); 
```

你根据需求，绘制出自己需要的 Path，就能定制出你要的速度模型。

不过要注意，这条 Path 描述的其实是一个 y = f(x) (0 ≤ x ≤ 1) （y 为动画完成度，x 为时间完成度）的曲线，所以同一段时间完成度上不能有两段不同的动画完成度（这个好理解吧？因为内容不能出现分身术呀），而且每一个时间完成度的点上都必须要有对应的动画完成度（因为内容不能在某段时间段内消失呀）。

除了上面的这些，Android 5.0 （API 21）引入了三个新的 Interpolator 模型，并把它们加入了 support v4 包中。这三个新的 Interpolator 每个都和之前的某个已有的 Interpolator 规则相似，只有略微的区别。

* **FastOutLinearInInterpolator**  
加速运动。

* **FastOutSlowInInterpolator**  
先加速再减速。

* **LinearOutSlowInInterpolator**  
持续减速。

### 3.设置监听器

给动画设置监听器，可以在关键时刻得到反馈，从而及时做出合适的操作，例如在动画的属性更新时同步更新其他数据，或者在动画结束后回收资源等。

设置监听器的方法， ViewPropertyAnimator 和 ObjectAnimator 略微不一样： ViewPropertyAnimator 用的是 setListener() 和 setUpdateListener() 方法，可以设置一个监听器，要移除监听器时通过 set[Update]Listener(null) 填 null 值来移除；而 ObjectAnimator 则是用 addListener() 和 addUpdateListener() 来添加一个或多个监听器，移除监听器则是通过 remove[Update]Listener() 来指定移除对象。

另外，由于 ObjectAnimator 支持使用 pause() 方法暂停，所以它还多了一个 addPauseListener() / removePauseListener() 的支持；而 ViewPropertyAnimator 则独有 withStartAction() 和 withEndAction() 方法，可以设置一次性的动画开始或结束的监听。

#### 3.1 ViewPropertyAnimator.setListener() / ObjectAnimator.addListener()

这两个方法的名称不一样，可以设置的监听器数量也不一样，但它们的参数类型都是 AnimatorListener，所以本质上其实都是一样的。 AnimatorListener 共有 4 个回调方法：

* 3.1.1 onAnimationStart(Animator animation)

当动画开始执行时，这个方法被调用。

* 3.1.2 onAnimationEnd(Animator animation)

当动画结束时，这个方法被调用。

* 3.1.3 onAnimationCancel(Animator animation)

当动画被通过 cancel() 方法取消时，这个方法被调用。

需要说明一下的是，就算动画被取消，onAnimationEnd() 也会被调用。所以当动画被取消时，如果设置了 AnimatorListener，那么 onAnimationCancel() 和 onAnimationEnd() 都会被调用。onAnimationCancel() 会先于 onAnimationEnd() 被调用。

* 3.1.4 onAnimationRepeat(Animator animation)

当动画通过 setRepeatMode() / setRepeatCount() 或 repeat() 方法重复执行时，这个方法被调用。

由于 ViewPropertyAnimator 不支持重复，所以这个方法对 ViewPropertyAnimator 相当于无效。

#### 3.2 ViewPropertyAnimator.setUpdateListener() / ObjectAnimator.addUpdateListener()

和上面 3.1 的两个方法一样，这两个方法虽然名称和可设置的监听器数量不一样，但本质其实都一样的，它们的参数都是 AnimatorUpdateListener。它只有一个回调方法：onAnimationUpdate(ValueAnimator animation)。

* 3.2.1 onAnimationUpdate(ValueAnimator animation)

当动画的属性更新时（不严谨的说，即每过 10 毫秒，动画的完成度更新时），这个方法被调用。

方法的参数是一个 ValueAnimator，ValueAnimator 是 ObjectAnimator 的父类，也是 ViewPropertyAnimator 的内部实现，所以这个参数其实就是 ViewPropertyAnimator 内部的那个 ValueAnimator，或者对于 ObjectAnimator 来说就是它自己本身。

ValueAnimator 有很多方法可以用，它可以查看当前的动画完成度、当前的属性值等等。

#### 3.3 ObjectAnimator.addPauseListener()
ObjectAnimator.pause() 后面讲

####3.4 ViewPropertyAnimator.withStartAction/EndAction()

这两个方法是 ViewPropertyAnimator 的独有方法。它们和 set/addListener() 中回调的 onAnimationStart() / onAnimationEnd() 相比起来的不同主要有两点：

* 1.withStartAction() / withEndAction() 是一次性的，在动画执行结束后就自动弃掉了，就算之后再重用 ViewPropertyAnimator 来做别的动画，用它们设置的回调也不会再被调用。而 set/addListener() 所设置的 AnimatorListener 是持续有效的，当动画重复执行时，回调总会被调用。

* 2.withEndAction() 设置的回调只有在动画正常结束时才会被调用，而在动画被取消时不会被执行。这点和 AnimatorListener.onAnimationEnd() 的行为是不一致的。

## TypeEvaluator

> 关于 ObjectAnimator，上期讲到可以用 ofInt() 来做整数的属性动画和用 ofFloat() 来做小数的属性动画。这两种属性类型是属性动画最常用的两种，不过在实际的开发中，可以做属性动画的类型还是有其他的一些类型。当需要对其他类型来做属性动画的时候，就需要用到 TypeEvaluator 了。  

> * 针对特殊类型的属性来做属性动画；
* 针对复杂的属性关系来做属性动画。

### ArgbEvaluator

```
ObjectAnimator animator = ObjectAnimator.ofInt(view, "color", 0xffff0000, 0xff00ff00);  
animator.setEvaluator(new ArgbEvaluator());  
animator.start();
```

另外，在 Android 5.0 （API 21） 加入了新的方法 ofArgb()，所以如果你的 minSdk 大于或者等于 21（，你可以直接用下面这种方式：

```
ObjectAnimator animator = ObjectAnimator.ofArgb(view, "color", 0xffff0000, 0xff00ff00);  
animator.start();  
```
![] (image/animator2.gif)

###自定义Evaluator

如果你对 ArgbEvaluator 的效果不满意，或者你由于别的什么原因希望写一个自定义的 TypeEvaluator，你可以这样写：

```
// 自定义 HslEvaluator
private class HsvEvaluator implements TypeEvaluator<Integer> {  
   float[] startHsv = new float[3];
   float[] endHsv = new float[3];
   float[] outHsv = new float[3];

   @Override
   public Integer evaluate(float fraction, Integer startValue, Integer endValue) {
       // 把 ARGB 转换成 HSV
       Color.colorToHSV(startValue, startHsv);
       Color.colorToHSV(endValue, endHsv);

       // 计算当前动画完成度（fraction）所对应的颜色值
       if (endHsv[0] - startHsv[0] > 180) {
           endHsv[0] -= 360;
       } else if (endHsv[0] - startHsv[0] < -180) {
           endHsv[0] += 360;
       }
       outHsv[0] = startHsv[0] + (endHsv[0] - startHsv[0]) * fraction;
       if (outHsv[0] > 360) {
           outHsv[0] -= 360;
       } else if (outHsv[0] < 0) {
           outHsv[0] += 360;
       }
       outHsv[1] = startHsv[1] + (endHsv[1] - startHsv[1]) * fraction;
       outHsv[2] = startHsv[2] + (endHsv[2] - startHsv[2]) * fraction;

       // 计算当前动画完成度（fraction）所对应的透明度
       int alpha = startValue >> 24 + (int) ((endValue >> 24 - startValue >> 24) * fraction);

       // 把 HSV 转换回 ARGB 返回
       return Color.HSVToColor(alpha, outHsv);
   }
}

ObjectAnimator animator = ObjectAnimator.ofInt(view, "color", 0xff00ff00, 0xff00ffff);  
// 使用自定义的 HslEvaluator
animator.setEvaluator(new HsvEvaluator());  
animator.start();  
```

![] (image/animator3.gif)

### ofObject()

借助于 TypeEvaluator，属性动画就可以通过 ofObject() 来对不限定类型的属性做动画了。方式很简单：

1. 为目标属性写一个自定义的 TypeEvaluator
2. 使用 ofObject() 来创建 Animator，并把自定义的 TypeEvaluator 作为参数填入

```
private class PointFEvaluator implements TypeEvaluator<PointF> {  
   PointF newPoint = new PointF();

   @Override
   public PointF evaluate(float fraction, PointF startValue, PointF endValue) {
       float x = startValue.x + (fraction * (endValue.x - startValue.x));
       float y = startValue.y + (fraction * (endValue.y - startValue.y));

       newPoint.set(x, y);

       return newPoint;
   }
}

ObjectAnimator animator = ObjectAnimator.ofObject(view, "position",  
        new PointFEvaluator(), new PointF(0, 0), new PointF(1, 1));
animator.start(); 
```

![] (image/animator4.gif)

另外在 API 21 中，已经自带了 PointFEvaluator 这个类，所以如果你的 minSdk 大于或者等于 21（，上面这个类你就不用写了，直接用就行了

TypeEvaluator 是本期的第一部分内容：针对特殊的属性来做属性动画，它可以让你「做到本来做不到的动画」。接下来是本期的第二部分内容：针对复杂的属性关系来做动画，它可以让你「能做到的动画做起来更简单」。

##PropertyValuesHolder 同一个动画中改变多个属性

很多时候，你在同一个动画中会需要改变多个属性，例如在改变透明度的同时改变尺寸。如果使用 ViewPropertyAnimator，你可以直接用连写的方式来在一个动画中同时改变多个属性：

```
view.animate()  
        .scaleX(1)
        .scaleY(1)
        .alpha(1);
```

![] (image/animator5.gif)

而对于 ObjectAnimator，是不能这么用的。不过你可以使用 PropertyValuesHolder 来同时在一个动画中改变多个属性。

```
PropertyValuesHolder holder1 = PropertyValuesHolder.ofFloat("scaleX",0， 1);  
PropertyValuesHolder holder2 = PropertyValuesHolder.ofFloat("scaleY", 0，1);  
PropertyValuesHolder holder3 = PropertyValuesHolder.ofFloat("alpha",0， 1);

ObjectAnimator animator = ObjectAnimator.ofPropertyValuesHolder(view, holder1, holder2, holder3)  
animator.start();  
```

PropertyValuesHolder 的意思从名字可以看出来，它是一个属性值的批量存放地。所以你如果有多个属性需要修改，可以把它们放在不同的 PropertyValuesHolder 中，然后使用 ofPropertyValuesHolder() 统一放进 Animator。这样你就不用为每个属性单独创建一个 Animator 分别执行了。

## AnimatorSet 多个动画配合执行
有的时候，你不止需要在一个动画中改变多个属性，还会需要多个动画配合工作，比如，在内容的大小从 0 放大到 100% 大小后开始移动。这种情况使用 PropertyValuesHolder 是不行的，因为这些属性如果放在同一个动画中，需要共享动画的开始时间、结束时间、Interpolator 等等一系列的设定，这样就不能有先后次序地执行动画了。

这就需要用到 AnimatorSet 了。

```
ObjectAnimator animator1 = ObjectAnimator.ofFloat(...);  
animator1.setInterpolator(new LinearInterpolator());  
ObjectAnimator animator2 = ObjectAnimator.ofInt(...);  
animator2.setInterpolator(new DecelerateInterpolator());

AnimatorSet animatorSet = new AnimatorSet();  
// 两个动画依次执行
animatorSet.playSequentially(animator1, animator2);  
animatorSet.start();  
```
![] (image/animator6.gif)

使用 playSequentially()，就可以让两个动画依次播放，而不用为它们设置监听器来手动为他们监管协作。

AnimatorSet 还可以这么用：

```
// 两个动画同时执行
animatorSet.playTogether(animator1, animator2);  
animatorSet.start(); 
``` 
以及这么用：

```
// 使用 AnimatorSet.play(animatorA).with/before/after(animatorB)
// 的方式来精确配置各个 Animator 之间的关系
animatorSet.play(animator1).with(animator2);  
animatorSet.play(animator1).before(animator2);  
animatorSet.play(animator1).after(animator2);  
animatorSet.start();  
```

##PropertyValuesHolders.ofKeyframe() 把同一个属性拆分

除了合并多个属性和调配多个动画，你还可以在 PropertyValuesHolder 的基础上更进一步，通过设置 Keyframe （关键帧），把同一个动画属性拆分成多个阶段。例如，你可以让一个进度增加到 100% 后再「反弹」回来。

```
// 在 0% 处开始
Keyframe keyframe1 = Keyframe.ofFloat(0, 0);  
// 时间经过 50% 的时候，动画完成度 100%
Keyframe keyframe2 = Keyframe.ofFloat(0.5f, 100);  
// 时间见过 100% 的时候，动画完成度倒退到 80%，即反弹 20%
Keyframe keyframe3 = Keyframe.ofFloat(1, 80);  
PropertyValuesHolder holder = PropertyValuesHolder.ofKeyframe("progress", keyframe1, keyframe2, keyframe3);

ObjectAnimator animator = ObjectAnimator.ofPropertyValuesHolder(view, holder);  
animator.start();  
```

![] (image/animator7.gif)

**总结**：

* 1. 使用 PropertyValuesHolder 来对多个属性同时做动画；
* 2. 使用 AnimatorSet 来同时管理调配多个动画；
* 3. PropertyValuesHolder 的进阶使用：使用 PropertyValuesHolder.ofKeyframe() 来把一个属性拆分成多段，执行更加精细的属性动画。

##ValueAnimator 最基本的轮子

额外简单说一下 ValuesAnimator。很多时候，你用不到它，只是在你使用一些第三方库的控件，而你想要做动画的属性却没有 setter / getter 方法的时候，会需要用到它。

除了 ViewPropertyAnimator 和 ObjectAnimator，还有第三个选择是 ValueAnimator。ValueAnimator 并不常用，因为它的功能太基础了。ValueAnimator 是 ObjectAnimator 的父类，实际上，ValueAnimator 就是一个不能指定目标对象版本的 ObjectAnimator。ObjectAnimator 是自动调用目标对象的 setter 方法来更新目标属性的值，以及很多的时候还会以此来改变目标对象的 UI，而 ValueAnimator 只是通过渐变的方式来改变一个独立的数据，这个数据不是属于某个对象的，至于在数据更新后要做什么事，全都由你来定，你可以依然是去调用某个对象的 setter 方法（别这么为难自己），也可以做其他的事，不管要做什么，都是要你自己来写的，ValueAnimator 不会帮你做。功能最少、最不方便，但有时也是束缚最少、最灵活。比如有的时候，你要给一个第三方控件做动画，你需要更新的那个属性没有 setter 方法，只能直接修改，这样的话 ObjectAnimator 就不灵了啊。怎么办？这个时候你就可以用 ValueAnimator，在它的 onUpdate() 里面更新这个属性的值，并且手动调用 invalidate()。

所以你看，ViewPropertyAnimator、ObjectAnimator、ValueAnimator 这三种 Animator，它们其实是一种递进的关系：从左到右依次变得更加难用，也更加灵活。但我要说明一下，它们的性能是一样的，因为 ViewPropertyAnimator 和 ObjectAnimator 的内部实现其实都是 ValueAnimator，ObjectAnimator 更是本来就是 ValueAnimator 的子类，它们三个的性能并没有差别。它们的差别只是使用的便捷性以及功能的灵活性。所以在实际使用时候的选择，只要遵循一个原则就行：尽量用简单的。能用 View.animate() 实现就不用 ObjectAnimator，能用 ObjectAnimator 就不用 ValueAnimator。