参考：
[android 自定义view教程] (http://www.gcssloop.com/customview/CustomViewIndex/)

本章可能会比较乱，暂时记录知识点，以后再整理格式

#View的坐标系
view的坐标系时相对于父控件而言的，如图：

![] (image/view1.jpg)

#MotionEvent中get和getRaw

```
event.getX();       //触摸点相对于其所在组件坐标系的坐标
event.getY();

event.getRawX();    //触摸点相对于屏幕默认坐标系的坐标
event.getRawY();
```
如图：

![] (image/view2.jpg
)