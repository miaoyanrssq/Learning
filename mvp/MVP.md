# MVP

## 1.1、MVP分层

总共分成三层

* a 、View: 视图层，对应xml文件与Activity/Fragment；
* b 、Presenter: 逻辑控制层，同时持有View和Model对象；
* c 、Model: 实体层，负责获取实体数据。
## 1.2、实现方式

总共四步

* a、根据项目需求，写一个 XXView 接口。然后让对应的 Activity/Fragment 实现这个接口。View 层基本搞定！
* b、编写 Model 层，主要就是网络数据请求了或者其他什么耗时操作，实现方式尽情发挥你的想象，但是最后一定需要用 Presenter 层定义的接口，回调给 Presenter 通知 View 层 更新数据。
* c、编写 Presenter 层，Presenter 层需要持有 View 层和 Model层的引用，并且实现 Presenter 层定义的回调接口。在回调接口中调用 View 层的代码 进行界面更新，最重要的是，有一个调用通过Model层的方法，在此方法中，调用 Model 层请求数据。
* d、回到View 层的Activity ，调用 Presenter 层获取数据。到此完成。
##1.3、原理图

![] (image/mvp1.png)

