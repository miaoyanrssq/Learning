# Android组件化开发
[TOC]
## 何为组件化开发
组件化开发就是将一个app分成多个模块，每个模块都是一个组件（Module），开发的过程中我们可以让这些组件相互依赖或者单独调试部分组件等，但是最终发布的时候是将这些组件合并统一成一个apk，这就是组件化开发。
## 组件化相关概念
### 模块化、组件化、插件化、热修复关系
**模块化与组件化**：模块化粒度更小，更侧重于重用，编译时间较长。而组件化粒度稍大于模块，各个组件可单独编译，更侧重于业务解耦。

**插件化与组件化**：插件化开发和组件化开发略有不用，插件化开发时将整个app拆分成很多模块，这些模块包括一个宿主和多个插件，每个模块都是一个apk（组件化的每个模块是个lib），最终打包的时候将宿主apk和插件apk分开或者联合打包。
### jar、arr、library、so等文件的关系

## 为何要用组件化
**常规开发模式：**
1. App不断迭代，业务越来越复杂，各个组件之间耦合度增加，开发维护成本增大。
2.  项目编译时会很卡很耗时，每次修改一处代码都要重新编译整个项目。
3.  耦合度高的代码，不便于做单元测试。

**组件化开发模式：**
1. 加快业务迭代速度，各个业务模块组件更加独立，不再出现业务耦合情况；
2. 稳定的公共模块采用依赖库方式，提供给各个业务线使用，减少重复开发和维护工作量；
3. 迭代频繁的业务模块采用组件方式，各业务研发可以互不干扰、提升协作效率，并控制产品质量；
4. 为新业务随时集成提供了基础，所有业务可上可下，灵活多变；
5. 降低团队成员熟悉项目的成本，降低项目的维护难度；
6. 加快编译速度，提高开发效率；
7. 系统级的控制力度细化到组件级的控制力度，一个复杂系统的构建最后就是组件集成的结果，每个组件都有自己独立的版本,可以独立的编译,测试,打包和部署。
8. 最重要的是可以降低公司的经营成本。

## 如何组件化开发
### 组件化开发结构划分
1. Main组件：属于业务组件，指定app启动页面、主界面。
2. Common组件：属于功能组件，支撑业务组件的基础，提供常用业务组件大都需要的功能（如提供网络请求）。
3. 功能组件：提供开发app的特定功能（如支付、crash日志统计、第三方登录分享）。
4. 业务组件：根据公司具体业务而独立开的一个工程。
5. app壳工程：负责管理各个业务组件和打包apk，没有具体业务功能。

集成模式：所有的组件被“app壳工程”依赖，组成一个完整的app。
组件模式：独立开发的组件，每个组件就是一个application。

Android组件化开发就是让各个组件在组件模式下并发开发，而在集成模式下，各组件又可变为arr包集成到app壳工程中，组成一个完整的app。简单的说就是开发时是 Application，发布时是 Library。

#### Common组件
1. Common组件的 AndroidManifest.xml 不是一张空表，这张表中声明了我们 Android应用用到的所有使用权限 uses-permission 和 uses-feature，放到这里是因为在组件开发模式下，所有业务组件就无需在自己的 AndroidManifest.xm 声明自己要用到的权限了。
2. Common组件的 build.gradle 需要统一依赖业务组件中用到的 第三方依赖库和jar包，例如我们用到的Router、Okhttp等等。
3. Common组件中封装了Android应用的 Base类和网络请求工具、图片加载工具等等，公用的 widget控件也应该放在Common 组件中；业务组件中都用到的数据也应放于Common组件中，例如保存到 SharedPreferences 和 DataBase 中的登陆数据；
4. Common组件的资源文件中需要放置项目**公用**的 Drawable、layout、sting、dimen、color和style 等等，另外项目中的 Activity 主题必须定义在 Common中，方便和 BaseActivity 配合保持整个Android应用的界面风格统一。
5. 在Common组件中我们封装了项目中用到的各种Base类，这些基类中就有BaseApplication 类。
BaseApplication 主要用于各个业务组件和app壳工程中声明的 Application 类继承用的，只要各个业务组件和app壳工程中声明的Application类继承了 BaseApplication，当应用启动时 BaseApplication 就会被动实例化。
> Common组件策略：这是一个被所有组件在组件模式和集成模式下都要依赖的module，所以一些常见的配置或公用的资源都应该在这里初始化。
### 组件模式和集成模式的转换
- application属性，可以独立运行的Android程序，也就是我们的APP；
```
apply plugin: ‘com.android.application’
```

- library属性，不可以独立运行，一般是Android程序依赖的库文件；
```
apply plugin: ‘com.android.library’
```
在Android项目中的任何一个build.gradle文件中都可以把gradle.properties中的常量读取出来。首先我们在gradle.properties中定义一个常量值 isDebug（是否是组件开发模式，true为是，false为否）：
```
# 是否是组件开发模式，true为是，false为否
isDebug = false
```
在业务组件中：
```
if (isDebug) {
    apply plugin: 'com.android.application'
} else {
    apply plugin: 'com.android.library'
}
```
当我们在组件模式开发时，业务组件应处于application属性，这时的业务组件就是一个 Android App，可以独立开发和调试；而当我们转换到集成模式开发时，业务组件应该处于 library 属性，这样才能被我们的“app壳工程”所依赖，组成一个具有完整功能的APP。

### 组件之间AndroidManifest的合并
在main文件夹下创建一个module文件夹用于存放组件开发模式下业务组件的 AndroidManifest.xml，而 AndroidStudio 生成的 AndroidManifest.xml 则依然保留，并用于集成开发模式下业务组件的表单；然后我们需要在业务组件的 build.gradle 中指定表单的路径。

- 在main子目录下新建module文件夹，并将组建的AndroidManifest.xml文件复制一份到此文件夹中。
- 在该业务组件android{}内添加sourceSets{...}内的代码，如下图
![Alt text](./1516613272912.png)
> 注意：不要写成Manifest，而是manifest。

集成模式下，业务组件的表单是绝对不能拥有自己的 Application 和 launch 的 Activity的，也不能声明APP名称、图标等属性。
### 全局Context的获取及组件数据初始化
上图中，对于集成开发模式下，会排除debug文件夹内的所有Java文件。这样可以区别集成开发模式/组件开发模式下对Application的配置。

![Alt text](./1516615625080.png)

我们在Java文件夹下创建一个 debug 文件夹，用于存放不会在业务组件中引用的类，例如Application 。
> 业务组件不能在集成开发模式下拥有自己的 Application，在组件开发模式下必须拥有自己的Application。

### 统一各Moudle依赖开源库版本
我们在系统级别gradle里进行版本号的统一配置，如图：
![Alt text](./1516608712870.png)
源码如下：
```
ext {
    minSdkVersion = 14
    targetSdkVersion = 26
    compileSdkVersion = 26
    buildToolsVersion = "26.0.2"

    butterknife = "8.8.1"
    butterknifeCompiler = "8.8.1"

    appcompatV7 = "26.+"
    constraintLayout = "1.0.2"
}
```
在各个moudle里如下配置：
![Alt text](./1516608816498.png)
![Alt text](./1516608894110.png)
### 解决资源id冲突
在合并多个组件到主工程中时,可能会出现资源引用冲突，最简单的方式是通过实现约定资源前缀名(resourcePrefix)来避免,需要在组件的gradle脚本中配置：
```
android {
    resourcePrefix "mine_"
    compileSdkVersion rootProject.ext.compileSdkVersion
    buildToolsVersion rootProject.ext.buildToolsVersion
...
}
```
一旦配置resourcePrefix，所有的资源必须以该前缀名开头。比如上面配置了前缀名为“mine_ ”，**那么该组件所有的资源名都要加上该前缀**，如：mine_save_bt。
### 组件间的通信
#### 为什么需要路由
Android系统已经给我们提供了api来做页面跳转，比如startActivity，为什么还需要路由框架呢？我们来简单分析下路由框架存在的意义：
- 由于组件化开发后，各个组件之间没有依赖关系，之间的通信和跳转不能再采用activity显示跳转去完成了。
- Activity隐式跳转是通过Intent匹配规则来实现，批量写起来繁琐易出错，不易定位问题。因此采用路由框架是最优方案。




#### Router在多进程开发的情况下通信	
**多进程开发的好处**：
- 提高各个进程的稳定性，单一进程崩溃后不影响整个程序。
- 对于内存的时候更可控，可以通过手工释放进程，达到内存优化目的。
- 基于独立的JVM，各个模块可以充分解耦。
- 只保留daemon进程的情况下，会使应用存活时间更长，不容易被回收掉。

Router是JVM级别的单例模式，并不支持跨进程访问。
**解决办法**：



## 引用
[Android组件化方案](http://www.uml.org.cn/mobiledev/201707133.asp)
[总结Android模块化的一些知识点](http://www.androidchina.net/7044.html)
[Android架构思考(模块化、多进程)](http://blog.spinytech.com/2016/12/28/android_modularization/)
[从零开始的Android新项目11 - 组件化实践（1）](http://blog.zhaiyifan.cn/2016/10/20/android-new-project-from-0-p11/)
[Android 模块化探索与实践](https://zhuanlan.zhihu.com/p/26744821)
[Android 路由框架ARouter最佳实践](http://blog.csdn.net/zhaoyanjun6/article/details/76165252)