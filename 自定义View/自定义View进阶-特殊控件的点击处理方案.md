#自定义View进阶-特殊控件的点击处理方案

##特殊形状控件
在通常的情况下，自定义 View 直接使用系统的事件体系处理就行，我们也不需要特殊处理，然而当一些特殊的控件出现的时候，麻烦就来了，举个栗子：
![] (image/click1.jpg)

##特殊形状控件点击区域判断

要进行特殊形状的点击判断，要用到一个之前没有使用过的类：`Region`。

Region 直接翻译的意思是 地域，区域。**在此处应该是区域的意思**。它和 Path 有些类似，但 Path 可以是不封闭图形，而 Region 总是封闭的。可以通过 `setPath`方法将 Path 转换为 Region。

本文中我们重点要使用到的是 Region 中的 `contains` 方法，这个方法可以判断一个点是否包含在该区域内。

接下来是一个简单的示例，**判断手指是否是在圆形区域内按下**:
![] (image/click2.jpg)

代码：

```
public class RegionClickView extends CustomView {
    Region circleRegion;
    Path circlePath;

    public RegionClickView(Context context) {
        super(context);
        mDeafultPaint.setColor(0xFF4E5268);
        circlePath = new Path();
        circleRegion = new Region();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
      	// ▼在屏幕中间添加一个圆
        circlePath.addCircle(w/2, h/2, 300, Path.Direction.CW);
        // ▼将剪裁边界设置为视图大小
        Region globalRegion = new Region(-w, -h, w, h);
        // ▼将 Path 添加到 Region 中
        circleRegion.setPath(circlePath, globalRegion);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                int x = (int) event.getX();
                int y = (int) event.getY();

                // ▼点击区域判断
                if (circleRegion.contains(x,y)){
                    Toast.makeText(this.getContext(),"圆被点击",Toast.LENGTH_SHORT).show();
                }
                break;
        }
        return true;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        // ▼注意此处将全局变量转化为局部变量，方便 GC 回收 canvas
        Path circle = circlePath;
        // 绘制圆
        canvas.drawPath(circle,mDeafultPaint);
    }
}
```

>代码中比较重要的内容都用 ▼ 符号标记出来了。

上述代码非常简单，就是创建了个 Path 并在其中添加圆形，之后将 Path 设置到 Region 中，当手指在屏幕上按下的时候判断手指位置是否在 Region 区域内。

##画布变换后坐标转换问题

还是本文一开始的例子，绘制一个上下左右选择按键，这个控件是上下左右对称的，熟悉我代码风格的小伙伴都知道，如果遇上这种问题，我肯定是要将坐标系平移到这个控件中心的，这样数据比较好计算，然而进行画布变换操作会产生一个新问题：**手指触摸的坐标系和画布坐标系不统一，就可能引起手指触摸位置和绘制位置不统一。**

解决方案：  转换坐标。

1.使用全局坐标系  
2使用逆矩阵的mapPoingts

```
@Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getActionMasked()){
            case MotionEvent.ACTION_DOWN:
            case MotionEvent.ACTION_MOVE:
                // ▼ 注意此处使用 getRawX，而不是 getX
                down_x = event.getRawX();
                down_y = event.getRawY();
                invalidate();
                break;

            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.ACTION_UP:
                down_x = down_y = -1;
                invalidate();
                break;
        }

        return true;
    }

```

```
float[] pts = {down_x, down_y};
// ▼注意画布平移
canvas.translate(mViewWidth/2, mViewHeight/2);          
if (pts[0] == -1 && pts[1] == -1) return;    // 如果没有就返回
// ▼ 获得当前矩阵的逆矩阵
Matrix invertMatrix = new Matrix();
canvas.getMatrix().invert(invertMatrix);

// ▼ 使用 mapPoints 将触摸位置转换为画布坐标
invertMatrix.mapPoints(pts);
// 在触摸位置绘制一个小圆
canvas.drawCircle(pts[0],pts[1],20,mDeafultPaint);
```

核心部分就两点：

```
// ▼ 注意此处使用 getRawX，而不是 getX
down_x = event.getRawX();
down_y = event.getRawY();

// -------------------------------------

// ▼ 获得当前矩阵的逆矩阵
Matrix invertMatrix = new Matrix();
canvas.getMatrix().invert(invertMatrix);

// ▼ 使用 mapPoints 将触摸位置转换为画布坐标
invertMatrix.mapPoints(pts);
```
**原理嘛，其实非常简单，我们在画布上正常的绘制，需要将画布坐标系转换为全局坐标系后才能真正的绘制内容。所以我们反着来，将获得到的全局坐标系坐标使用当前画布的逆矩阵转化一下，就转化为当前画布的坐标系坐标了，如果对 Matrix原理 和 Matrix详解 理解了，即便我不说你们也肯定会想到这个方案的。**

##仿遥控器按钮代码示例

![] (image/click3.jpg)

代码：

```
public class RemoteControlMenu extends CustomView {
    Path up_p, down_p, left_p, right_p, center_p;
    Region up, down, left, right, center;

    Matrix mMapMatrix = null;

    int CENTER = 0;
    int UP = 1;
    int RIGHT = 2;
    int DOWN = 3;
    int LEFT = 4;
    int touchFlag = -1;
    int currentFlag = -1;

    MenuListener mListener = null;

    int mDefauColor = 0xFF4E5268;
    int mTouchedColor = 0xFFDF9C81;


    public RemoteControlMenu(Context context) {
        this(context, null);
    }

    public RemoteControlMenu(Context context, AttributeSet attrs) {
        super(context, attrs);

        up_p = new Path();
        down_p = new Path();
        left_p = new Path();
        right_p = new Path();
        center_p = new Path();

        up = new Region();
        down = new Region();
        left = new Region();
        right = new Region();
        center = new Region();

        mDeafultPaint.setColor(mDefauColor);
      	mDeafultPaint.setAntiAlias(true);

        mMapMatrix = new Matrix();

    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mMapMatrix.reset();
		
      	// 注意这个区域的大小
        Region globalRegion = new Region(-w, -h, w, h);
        int minWidth = w > h ? h : w;
        minWidth *= 0.8;

        int br = minWidth / 2;
        RectF bigCircle = new RectF(-br, -br, br, br);

        int sr = minWidth / 4;
        RectF smallCircle = new RectF(-sr, -sr, sr, sr);

        float bigSweepAngle = 84;
        float smallSweepAngle = -80;

        // 根据视图大小，初始化 Path 和 Region
        center_p.addCircle(0, 0, 0.2f * minWidth, Path.Direction.CW);
        center.setPath(center_p, globalRegion);

        right_p.addArc(bigCircle, -40, bigSweepAngle);
        right_p.arcTo(smallCircle, 40, smallSweepAngle);
        right_p.close();
        right.setPath(right_p, globalRegion);

        down_p.addArc(bigCircle, 50, bigSweepAngle);
        down_p.arcTo(smallCircle, 130, smallSweepAngle);
        down_p.close();
        down.setPath(down_p, globalRegion);

        left_p.addArc(bigCircle, 140, bigSweepAngle);
        left_p.arcTo(smallCircle, 220, smallSweepAngle);
        left_p.close();
        left.setPath(left_p, globalRegion);

        up_p.addArc(bigCircle, 230, bigSweepAngle);
        up_p.arcTo(smallCircle, 310, smallSweepAngle);
        up_p.close();
        up.setPath(up_p, globalRegion);

    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        float[] pts = new float[2];
        pts[0] = event.getRawX();
        pts[1] = event.getRawY();
        mMapMatrix.mapPoints(pts);

        int x = (int) pts[0];
        int y = (int) pts[1];

        switch (event.getActionMasked()) {
            case MotionEvent.ACTION_DOWN:
                touchFlag = getTouchedPath(x, y);
                currentFlag = touchFlag;
                break;
            case MotionEvent.ACTION_MOVE:
                currentFlag = getTouchedPath(x, y);
                break;
            case MotionEvent.ACTION_UP:
                currentFlag = getTouchedPath(x, y);
            	// 如果手指按下区域和抬起区域相同且不为空，则判断点击事件
                if (currentFlag == touchFlag && currentFlag != -1 && mListener != null) {
                    if (currentFlag == CENTER) {
                        mListener.onCenterCliched();
                    } else if (currentFlag == UP) {
                        mListener.onUpCliched();
                    } else if (currentFlag == RIGHT) {
                        mListener.onRightCliched();
                    } else if (currentFlag == DOWN) {
                        mListener.onDownCliched();
                    } else if (currentFlag == LEFT) {
                        mListener.onLeftCliched();
                    }
                }
                touchFlag = currentFlag = -1;
                break;
            case MotionEvent.ACTION_CANCEL:
                touchFlag = currentFlag = -1;
                break;
        }

        invalidate();
        return true;
    }

  	// 获取当前触摸点在哪个区域
    int getTouchedPath(int x, int y) {
        if (center.contains(x, y)) {
            return 0;
        } else if (up.contains(x, y)) {
            return 1;
        } else if (right.contains(x, y)) {
            return 2;
        } else if (down.contains(x, y)) {
            return 3;
        } else if (left.contains(x, y)) {
            return 4;
        }
        return -1;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.translate(mViewWidth / 2, mViewHeight / 2);

        // 获取测量矩阵(逆矩阵)
        if (mMapMatrix.isIdentity()) {
            canvas.getMatrix().invert(mMapMatrix);
        }

      	// 绘制默认颜色
        canvas.drawPath(center_p, mDeafultPaint);
        canvas.drawPath(up_p, mDeafultPaint);
        canvas.drawPath(right_p, mDeafultPaint);
        canvas.drawPath(down_p, mDeafultPaint);
        canvas.drawPath(left_p, mDeafultPaint);

      	// 绘制触摸区域颜色
        mDeafultPaint.setColor(mTouchedColor);
        if (currentFlag == CENTER) {
            canvas.drawPath(center_p, mDeafultPaint);
        } else if (currentFlag == UP) {
            canvas.drawPath(up_p, mDeafultPaint);
        } else if (currentFlag == RIGHT) {
            canvas.drawPath(right_p, mDeafultPaint);
        } else if (currentFlag == DOWN) {
            canvas.drawPath(down_p, mDeafultPaint);
        } else if (currentFlag == LEFT) {
            canvas.drawPath(left_p, mDeafultPaint);
        }
        mDeafultPaint.setColor(mDefauColor);
    }

    public void setListener(MenuListener listener) {
        mListener = listener;
    }

   	// 点击事件监听器
    public interface MenuListener {
        void onCenterCliched();

        void onUpCliched();

        void onRightCliched();

        void onDownCliched();

        void onLeftCliched();
    }
}
```

**运行效果:**

当手指在某一区域活动时，该区域会高亮显示，如果注册了监听器，点击某一区域会触发监听器回调。

##关于硬件加速的问题

Matrix 的作用： **Matrix作用就是坐标映射。** 
其核心功能就是将单个 View 的坐标系转化为屏幕(物理)坐标系，虽然转换一次费不了多少时间，但是当执行动画效果等需要大量快速重绘的情况下，耗费的时间就需要考量一下了，于是乎，硬件加速干了一件非常精明的事情，**把所有画布坐标系都设置为屏幕(物理)坐标系，**之后在 View 绘制区域设置一个遮罩，保证绘制内容不会超过 View 自身的大小，**这样就直接跳过坐标转换过程，可以节省坐标系之间数值转换耗费的时间。**因此导致了以下问题：

* 开启硬件加速情况下 event.getX() 和 不开启情况下 event.getRawX() 等价，获取到的是屏幕(物理)坐标 (本文的锅)。
* 开启硬件加速情况下 event.getRawX() 数值是一个错误数值，因为本身就是全局的坐标又叠加了一次 View 的偏移量，所以肯定是不正确的 (本文的锅)。
* 从 Canvas 获取到的 Matrix 是全局的，默认情况下 x,y 偏移量始终为0，因此你不能从这里拿到当前 View 的偏移量 ( Matrix系列文章中的锅 )。
* 由于其使用的是遮罩来控制绘制区域，所以如果重绘 path 时，如果 path 区域变大，但没有执行单步操作会导致 path 绘制不完整或者看起来比较奇怪 (Path系列文章中的锅)。  

很显然，这个硬件加速有点6，制造了各种锅想让我来背，然而智慧的我早已看穿一切，默默的把硬件加速关闭了，因为我不知道它还有多少锅没亮出来。


个人建议：

* APP全局关闭硬件加速。
* 针对动画较多的 Activity 或者 View 单独开启硬件加速。
* 如果应用要兼容到 3.0 以下，不要使用硬件加速的特性，或者进行兼容处理。
* 如果 自定义View 出现与绘图相关的异常，请务必检查一下硬件加速。

##多点触控

[参考链接] (http://www.gcssloop.com/customview/multi-touch)
