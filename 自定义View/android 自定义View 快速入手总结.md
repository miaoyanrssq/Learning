#android 自定义View

>看了很多关于自定义view的文章，但是实践的很少，每次遇到一个漂亮的效果想要实现一下，却发现无从下手。  
>本文旨在让新手掌握自定义view的过程，快速写出一些简单的view以及动画效果。  
>推荐本人看过的最好的最全面的关于自定义view的文章:[自定义view](http://hencoder.com/ui-1-1/)


##关键
首先要认清楚，自定义view，ondraw（）里面的是图，<mark>静态</mark>的图,而且这个绘制在我们视觉效果上是没有过程的，一瞬间完成。所以所有的动态效果，都是我们通过动画，常用的如ValueAnimator，来动态改变绘制的参数，然后invalidate（），刷新画布，重新绘制的。

##关键步骤

1、extend <mark>View</mark>  
先继承某个view  

2、构造函数  
最好三个都写上，不然可能会报错

```
public MyView2(Context context) {
        this(context, null);

}

public MyView2(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
}

public MyView2(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        paint = new Paint();
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.FILL);
        paint.setAlpha(100);
        paint.setAntiAlias(true);
        paint.setStrokeWidth(10);

        animator = ValueAnimator.ofInt(10, 300).setDuration(2000);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                radus = (int) animation.getAnimatedValue();
                invalidate();
            }
        });
        animator.setRepeatMode(ValueAnimator.REVERSE);
        animator.setRepeatCount(10);
        animator.start();
}
```
构造函数要做的事是，<mark>初始化</mark>，<mark>初始化</mark>，<mark>初始化</mark>，所有的东西都在这里初始化  

3、onDraw()  
<mark>绘制</mark>，绘制静态图。

```
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    canvas.drawCircle(300+ radus, 300 + radus, radus, paint);

}
```

在这里我画了一个圆，如果想要动画，那就把绘制参数设置为全局变量，然后在animator的监听中改变参数，然后invalidate（），图就动起来了。参考构造函数里面animator的监听

```
animator = ValueAnimator.ofInt(10, 300).setDuration(2000);
animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
        @Override
        public void onAnimationUpdate(ValueAnimator animation) {
            radus = (int) animation.getAnimatedValue();
            invalidate();
        }
    });
 animator.setRepeatMode(ValueAnimator.REVERSE);
 animator.setRepeatCount(10);
 animator.start();
```
4、onMeasure()  
测量布局的宽高  

5、onLayout()  
布局，就是放在什么位置，在View中没用，ViewGroup中摆放子view的，具体参考文首的连接 

##总结
到这里，整个自定义view的框架全部完成了，至少普通的view和动画，都难不倒我们了，如果想要做出炫酷的效果，就需要各种算法控制的东西了，这个就要自己学了。

