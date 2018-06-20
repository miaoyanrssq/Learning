# 自定义View进阶-手势检测(GestureDecetor)

> GestureDetector 可以使用 MotionEvents 检测各种手势和事件。GestureDetector.OnGestureListener 是一个回调方法，在发生特定的事件时会调用 Listener 中对应的方法回调。这个类只能用于检测触摸事件的 MotionEvent

>如何使用：

> * 创建一个 GestureDetector 实例。
* 在onTouchEvent（MotionEvent）方法中，确保调用 GestureDetector 实例的 onTouchEvent（MotionEvent）。回调中定义的方法将在事件发生时执行。
* 如果侦听 onContextClick（MotionEvent），则必须在 View 的 onGenericMotionEvent（MotionEvent）中调用 GestureDetector OnGenericMotionEvent（MotionEvent）。

GestureDetector 本身的方法比较少，使用起来也非常简单，下面让我们先看一下它的简单使用示例，分解开来大概需要三个步骤。

```
// 1.创建一个监听回调
SimpleOnGestureListener listener = new SimpleOnGestureListener() {
    @Override public boolean onDoubleTap(MotionEvent e) {
        Toast.makeText(MainActivity.this, "双击666", Toast.LENGTH_SHORT).show();
        return super.onDoubleTap(e);
    }
};

// 2.创建一个检测器
final GestureDetector detector = new GestureDetector(this, listener);

// 3.给监听器设置数据源
view.setOnTouchListener(new View.OnTouchListener() {
    @Override public boolean onTouch(View v, MotionEvent event) {
        return detector.onTouchEvent(event);
    }
});
```

##构造函数

![] (image/gestureDetector.png)

第 1 种构造函数里面需要传递两个参数，上下文(Context) 和 手势监听器(OnGestureListener)，这个很容易理解，就不再过多叙述，上面的例子中使用的就是这一种。

第 2 种构造函数则需要多传递一个 Handler 作为参数，这个有什么作用呢？其实作用也非常简单，这个 Handler 主要是为了给 GestureDetector 提供一个 Looper。

在通常情况下是不需这个 Handler 的，因为它会在内部自动创建一个 Handler 用于处理数据，如果你在主线程中创建 GestureDetector，那么它内部创建的 Handler 会自动获得主线程的 Looper，然而如果你在一个没有创建 Looper 的子线程中创建 GestureDetector 则需要传递一个带有 Looper 的 Handler 给它，否则就会因为无法获取到 Looper 导致创建失败。

第 2 种构造函数使用方式如下(下面是两种在子线程中创建 GestureDetector 的方法)：

```
// 方式一、在主线程创建 Handler
final Handler handler = new Handler();
new Thread(new Runnable() {
    @Override public void run() {
        final GestureDetector detector = new GestureDetector(MainActivity.this, new
                GestureDetector.SimpleOnGestureListener() , handler);
        // ... 省略其它代码 ...
    }
}).start();

// 方式二、在子线程创建 Handler，并且指定 Looper
new Thread(new Runnable() {
    @Override public void run() {
        final Handler handler = new Handler(Looper.getMainLooper());
        final GestureDetector detector = new GestureDetector(MainActivity.this, new
                GestureDetector.SimpleOnGestureListener() , handler);
        // ... 省略其它代码 ...
    }
}).start();
```

当然了，使用其它创建 Handler 的方式也是可以的，重点传递的 Handler 一定要有 Looper，敲黑板，重点是 Handler 中的 Looper。假如子线程准备了 Looper 那么可以直接使用第 1 种构造函数进行创建，如下：

```
new Thread(new Runnable() {
    @Override public void run() {
        Looper.prepare(); // <- 重点在这里
        final GestureDetector detector = new GestureDetector(MainActivity.this, new
                GestureDetector.SimpleOnGestureListener());
        // ... 省略其它代码 ...
    }
}).start();
```

## 手势监听器

既然是手势检测，自然要在对应的手势出现的时候通知调用者，最合适的自然是事件监听器模式。目前 GestureDetecotr 有四种监听器。

![] (image/gestureDetector2.png)

### OnContextClickListener

由于 OnContextClickListener 主要是用于检测外部设备按钮的，关于它需要注意一点，如果侦听 onContextClick(MotionEvent)，则必须在 View 的 onGenericMotionEvent(MotionEvent)中调用 GestureDetector 的 OnGenericMotionEvent(MotionEvent)。

### OnDoubleTapListener

这个很明显就是用于检测双击事件的，它有三个回调接口，分别是 onDoubleTap、onDoubleTapEvent 和 onSingleTapConfirmed。

**onDoubleTap 与 onSingleTapConfirmed**

**如果你只想监听双击事件，那么只用关注 onDoubleTap 就行了，如果你同时要监听单击事件则需要关注 onSingleTapConfirmed 这个回调函数。**

有人可能会有疑问，监听单击事件为什么要使用 onSingleTapConfirmed，使用 OnClickListener 不行吗？从理论上是可行的，但是我并不推荐这样使用，主要有两个原因：  
1.它们两个是存在一定冲突的，如果你看过 事件分发机制详解 就会知道，如果想要两者同时被触发，则 setOnTouchListener 不能消费事件，如果 onTouchListener 消费了事件，就可能导致 OnClick 无法正常触发。  
2.需要同时监听单击和双击，则说明单击和双击后响应逻辑不同，然而使用 OnClickListener 会在双击事件发生时触发两次，这显然不是我们想要的结果。而使用 onSingleTapConfirmed 就不用考虑那么多了，你完全可以把它当成单击事件来看待，而且在双击事件发生时，onSingleTapConfirmed 不会被调用，这样就不会引发冲突。

如果你需要同时监听两种点击事件可以这样写：

```
GestureDetector detector = new GestureDetector(this, new GestureDetector
        .SimpleOnGestureListener() {
    @Override public boolean onSingleTapConfirmed(MotionEvent e) {
        Toast.makeText(MainActivity.this, "单击", Toast.LENGTH_SHORT).show();
        return false;
    }
    @Override public boolean onDoubleTap(MotionEvent e) {
        Toast.makeText(MainActivity.this, "双击", Toast.LENGTH_SHORT).show();
        return false;
    }
});
```

关于 onSingleTapConfirmed 原理也非常简单，这一个回调函数在单击事件发生后300ms后触发(注意，不是立即触发的)，只有在确定不会有后续的事件后，既当前事件肯定是单击事件才触发 onSingleTapConfirmed，所以在进行点击操作时，onDoubleTap 和 onSingleTapConfirmed 只会有一个被触发，也就不存在冲突了。

**onDoubleTapEvent**

**有些细心的小伙伴可能注意到还有一个 onDoubleTapEvent 回调函数，它是干什么的呢？它在双击事件确定发生时会对第二次按下产生的 MotionEvent 信息进行回调。**


至于为什么要存在这样的回调，就要涉及到另一个比较细致的问题了，那就是 onDoubleTap 的触发时间，如果你在这些函数被调用时打印一条日志，那么你会看到这样的信息：

```
GCS-LOG: onDoubleTap
GCS-LOG: onDoubleTapEvent - down
GCS-LOG: onDoubleTapEvent - move
GCS-LOG: onDoubleTapEvent - move
GCS-LOG: onDoubleTapEvent - up
```
通过观察这些信息你会发现它们的调用顺序非常有趣，首先是 onDoubleTap 被触发，之后依次触发 onDoubleTapEvent 的 down、move、up 等信息，为什么说它们有趣呢？是因为这样的调用顺序会引发两种猜想，第一种猜想是 onDoubleTap 是在第二次手指抬起(up)后触发的，而 onDoubleTapEvent 是一种延时回调。第二种猜想则是 onDoubleTap 在第二次手指按下(dowm)时触发，onDoubleTapEvent 是一种实时回调。

通过测试和观察源码发现第二种猜想是正确的，因为第二次按下手指时，即便不抬起也会触发 onDoubleTap 和 onDoubleTapEvent 的 down，而且源码中逻辑也表明 onDoubleTapEvent 是一种实时回调。

这就引发了另一个问题，双击的触发时间，虽然这是一个细微到很难让人注意到的问题，假如说我们想要在第二次按下抬起后才判定这是一个双击操作，触发后续的内容，则不能使用 onDoubleTap 了，需要使用 onDoubleTapEvent 来进行更细微的控制，如下：

```
final GestureDetector detector = new GestureDetector(MainActivity.this, new GestureDetector.SimpleOnGestureListener() {
    @Override public boolean onDoubleTap(MotionEvent e) {
        Logger.e("第二次按下时触发");
        return super.onDoubleTap(e);
    }

    @Override public boolean onDoubleTapEvent(MotionEvent e) {
        switch (e.getActionMasked()) {
            case MotionEvent.ACTION_UP:
                Logger.e("第二次抬起时触发");
                break;
        }
        return super.onDoubleTapEvent(e);
    }
});
```
如果你不需要控制这么细微的话，忽略即可。

###OnGestureListener

这个是手势检测中较为核心的一个部分了，主要检测以下类型事件：按下(Down)、 一扔(Fling)、长按(LongPress)、滚动(Scroll)、触摸反馈(ShowPress) 和 单击抬起(SingleTapUp)。

**onDown**

```
@Override 
public boolean onDown(MotionEvent e) {
    return true;
}
```
看过前面的文章应该知道，down 在事件分发体系中是一个较为特殊的事件，为了保证事件被唯一的 View 消费，哪个 View 消费了 down 事件，后续的内容就会传递给该 View。如果我们想让一个 View 能够接收到事件，有两种做法：

1、让该 View 可以点击，因为可点击状态会默认消费 down 事件。

2、手动消费掉 down 事件。

由于图片、文本等一些控件默认是不可点击的，所以我们要么声明它们的 clickable 为 true，要么在发生 down 事件是返回 true。所以 onDown 在这里的作用就很明显了，就是为了保证让该控件能拥有消费事件的能力，以接受后续的事件。

**onFling**
Fling 中文直接翻译过来就是一扔、抛、甩，最常见的场景就是在 ListView 或者 RecyclerView 上快速滑动时手指抬起后它还会滚动一段时间才会停止。onFling 就是检测这种手势的。

```
@Override
public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float
        velocityY) {
    return super.onFling(e1, e2, velocityX, velocityY);
}
```

![] (image/gestureDetector3.png)

我们可以通过 e1 和 e2 获取到手指按下和抬起时的坐标、时间等相关信息，通过 velocityX 和 velocityY 获取到在这段时间内的运动速度，单位是像素／秒(即 1 秒内滑动的像素距离)。

**onLongPress**

这个是检测长按事件的，即手指按下后不抬起，在一段时间后会触发该事件。

```
@Override 
public void onLongPress(MotionEvent e) {
}
```

**onScroll**

onScroll 就是监听滚动事件的，它看起来和 onFling 比较像，不同的是，onSrcoll 后两个参数不是速度，而是滚动的距离。

```
@Override
public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float 
        distanceY) {
    return super.onScroll(e1, e2, distanceX, distanceY);
}
```
![] (image/gestureDetector4.png)

**onShowPress**

它是用户按下时的一种回调，主要作用是给用户提供一种视觉反馈，可以在监听到这种事件时可以让控件换一种颜色，或者产生一些变化，告诉用户他的动作已经被识别。

不过这个消息和 onSingleTapConfirmed 类似，也是一种延时回调，延迟时间是 180 ms，假如用户手指按下后立即抬起或者事件立即被拦截，时间没有超过 180 ms的话，这条消息会被 remove 掉，也就不会触发这个回调。

```
@Override 
public void onShowPress(MotionEvent e) {
}
```
**onSingleTapUp**

```
@Override 
public boolean onSingleTapUp(MotionEvent e) {
    return super.onSingleTapUp(e);
}
```
这个也很容易理解，就是用户单击抬起时的回调

###SimpleOnGestureListener
这个里面并没有什么内容，只是对上面三种 Listener 的空实现，在上面的例子中使用的基本都是这监听器。因为它用起来更方便一点。

##相关方法

除了各类监听器之外，与 GestureDetector 相关的方法其实并不多，只有几个，下面来简单介绍一下

![] (image/gestureDetector5.png)


##ScaleGestureDetector

后续补充
