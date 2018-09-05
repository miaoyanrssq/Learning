# Jenkins服务器搭建（Mac）
> 本文蛀牙针对Android开发用户，默认使用Android Studio 开发
## 1、添加路径

1. 终端输入： `touch .bash_profile`
2. 终端输入：`open -e .bash_profile`
3. 添加Android SDK platform-tools的路径
4. 终端输入:`source .bash_profile`
5. 终端输入:`adb`（校验是否配置成功）

![] (image/jk1.png)

![] (image/jk2.png)

## 2、配置gtadle命令
一般Android开发者已经安装了。如果没有安装，使用命令`brew install gradle`安装。  
可以输入`gradle -version`检验是否安装成功。

**如何安装brew**  
终端输入：`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" `

## 3、安装jenkins
终端输入：`brew install jenkins`  
安装成功后输入`jenkins`启动jenkins  
然后就可以在本地浏览器输入`localhost:8080`即可访问了

## 4、jenkins环境配置

### 安装插件
Git Plugin  
Gradle Plugin  
Android Lint Plugin

![] (image/jk3.png)

![] (image/jk4.png)

### 环境变量配置

![] (image/jk5.png)

![] (image/jk6.png)

**至此jenkins的配置就结束了**  
下面是具体项目配置

# github的Android项目

## 1、新建一个自由风格的项目


![] (image/jk7.png)

## 2、 配置git地址

![] (image/jk8.png)

如果需要git权限，则Add相关git账户

## 3、立即构建

![] (image/jk9.jpg)

上面虽然编译成功了，但是你发现根本没有看到APK文件，所以还需要进行下面的配置：增加构建步骤，选择shell命令


![] (image/jk10.png)

![] (image/jk11.png)

这样就可以在`工作空间`查看apk了。  

# 如果需要动态改变参数，做如下配置

1. 添加阐述
![] (image/jk12.png)

2. 增加构建步骤

![] (image/jk13.png)

`echo "appVersionCode="$appVersionCode"\nappVersionName="$appVersionName"\nenv="$env"\nisDebug="$isDebug"\napp_name="$app_name>zbps.properties`

这样会生成zbps.properties文件，可以在项目的gradle插件中引用相关参数。

![] (image/jk14.png)

在build.gradle中引入插件就可以使用相关参数了

![] (image/jk15.png)


# 上传蒲公英

增加构建步骤，选择shell命令,输入`curl -F "file=@这里是你的APK文件位置" -F "uKey=蒲公英的key" -F "_api_key=蒲公英的api_KEY" https://www.pgyer.com/apiv1/app/upload`即可。


