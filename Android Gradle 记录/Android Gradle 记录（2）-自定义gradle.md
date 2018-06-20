# Android Gradle 记录（2）-自定义


## 使用共享库
Android 中包含在sdk库里的包，app可以直接使用，系统会自动链接它们，但是，有一些库是独立的，不会被系统自动链接，如果我们要使用的话，要单独进行生成，我们称之为共享库，如：`com.google.android.maps`、`android.test.runner`等。

在AndroidManifest中，这样指定：

```
<uses-library
	android:name="com.google.android.maps"
	android:required="true"
```
这样在安装apk的时候，系统会检测手机系统是否包含此共享库，因为设置了required=true，如果不包含，就不能安装app。

gradle中添加方法：

```
Android{
	useLibrary "org.apache.http.legacy"
}
```
最好在AndroidMnifest中配置一下`uses-library`, 以防出错。

## 批量修改生成apk的文件名

因为Android工程很多任务是动态创建和生成的，而且生成的时间比较靠后，所以无法通过project.tasks获取一个任务。  
可以通过Android提供的applicationVariants（仅适用于Android应用的Gradle插件）、libraryVariants（仅适用于Android库Gradle插件）、testVarients（以上两种都适用）。  
访问以上三种集合都会触发创建所有的任务，所以我们通过访问这些集合，修改生成apk的输出文件名，那么就会自动触发创建所有的任务，从而修改后的apk名字就会起作用。下面给出一个例子：

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'
    }
}

apply plugin: 'com.android.application'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.1"

    useLibrary 'org.apache.http.legacy'


    defaultConfig {
        applicationId "org.flysnow.app.example92"
        minSdkVersion 14
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            zipAlignEnabled true
        }
    }
    productFlavors {
        google {

        }
    }
    applicationVariants.all { variant ->
        variant.outputs.each { output ->
            if (output.outputFile != null && output.outputFile.name.endsWith('.apk')
                    &&'release'.equals(variant.buildType.name)) {
                def flavorName = variant.flavorName.startsWith("_") ? variant.flavorName.substring(1) : variant.flavorName
                def apkFile = new File(
                        output.outputFile.getParent(),
                        "Example92_${flavorName}_v${variant.versionName}_${buildTime()}.apk")
                output.outputFile = apkFile
            }
        }
    }
}

def buildTime() {
    def date = new Date()
    def formattedDate = date.format('yyyyMMdd')
    return formattedDate
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
}
```

## 动态生成版本信息

1. 统一版本号：  

创建一个version.gradle:

```
ext {
    appVersionCode =1
    appVersionName = "1.0.0"
}
```
ext{}块表明要为当前project创建扩展属性，，以供其他脚本引用。创建好后，在build.gradle中通过`apply from`引用：

```
apply from: 'version.gradle'

android{
	
	deafultConfig{
		
		versionCode appVersionCode
		versionName appVersionName
	}
}
```

2. 从git的tag中获取

前提，每次发包的时候git上打tag，tag内容为versionName，则tag的数量就为versionCode
代码如下：

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'
    }
}

apply plugin: 'com.android.application'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.1"


    defaultConfig {
        applicationId "org.flysnow.app.example93"
        minSdkVersion 14
        targetSdkVersion 23
        versionCode getAppVersionCode()
        versionName getAppVersionName()
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            zipAlignEnabled true
        }
    }
}
/**
 * 以git tag的数量作为其版本号
 * @return tag的数量
 */
def getAppVersionCode(){
    def stdout = new ByteArrayOutputStream()
    exec {
        commandLine 'git','tag','--list'
        standardOutput = stdout
    }
    return stdout.toString().split("\n").size()
}

/**
 * 从git tag中获取应用的版本名称
 * @return git tag的名称
 */
def getAppVersionName(){
    def stdout = new ByteArrayOutputStream()
    exec {
        commandLine 'git','describe','--abbrev=0','--tags'
        standardOutput = stdout
    }
    return stdout.toString().replaceAll("\n","")
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
}
```

### 隐藏签名文件信息

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'
    }
}

apply plugin: 'com.android.application'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.1"

    signingConfigs {
        def appStoreFile = System.getenv("STORE_FILE")
        def appStorePassword = System.getenv("STORE_PASSWORD")
        def appKeyAlias = System.getenv("KEY_ALIAS")
        def appKeyPassword = System.getenv("KEY_PASSWORD")

        //当不能从环境变量里获取到签名信息的时候，就使用debug模式的签名
        if(!appStoreFile||!appStorePassword||!appKeyAlias||!appKeyPassword){
            appStoreFile = "debug.keystore"
            appStorePassword = "android"
            appKeyAlias = "androiddebugkey"
            appKeyPassword = "android"
        }
        release {
            storeFile file(appStoreFile)
            storePassword appStorePassword
            keyAlias appKeyAlias
            keyPassword appKeyPassword
        }
    }

    defaultConfig {
        applicationId "org.flysnow.app.example94"
        minSdkVersion 14
        targetSdkVersion 23
        versionCode 1
        versionName '1.0.0'
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            zipAlignEnabled true
        }
    }
}
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
}
```
`System.getenv("STORE_FILE")`是获取系统的环境变量，我们需要在本地配置环境变量，类似sdk环境变量的配置，这样就能获取到本地的配置信息，如果没有获取到相关的信息，就使用系统debug的签名包。

## 动态配置AndroidManifest文件

举个例子：

```
<mata-data android:value="Channel ID" android: name = "UMENG_CHANNEL"/>
```
友盟等第三方包，会要求我们在AndroidManifest中指定渠道名称，上面的Channel ID 我们要替换成不同的渠道名称，如 google， baidu等。  
因此，我们可以在构建的时候，根据正在生成的不同渠道包开为其指定不同的渠道名。Android Gradle中提供了`manifestPlaceholder`、`manifest`占位符

```
android{
	compileSdkVersion 23
	……
	
	productFlavors{
		google{
	    	manifestPlaceholder.put("UMENG_CHANNEL", "google")
		}
		baidu{
	    	manifestPlaceholder.put("UMENG_CHANNEL", "baidu")
		}
		
	}
}
```
而`AndroidManistfest`文件中相应的位置用如下代替即可（${name}）

```
<meta-data android:value="${UMENG_CHANNEL}" android:name="UMENG_CHANNEL"/>

```

如果渠道很多，可以通过迭代的方式替换：

```
android{
	compileSdkVersion 23
	……
	
	productFlavors{
		google{}
		baidu{}
	}
	
	productFlavors.all{ flavor ->
		manifestPlaceholder.put("UMENG_CHANNEL", "name")
	}
}

```

### 自定义你的BuildConfig

```
/**
 * Automatically generated file. DO NOT MODIFY
 */
package com.example.gaoyangzhen.autoview;

public final class BuildConfig {
  public static final boolean DEBUG = Boolean.parseBoolean("true");
  public static final String APPLICATION_ID = "com.example.gaoyangzhen.autoview";
  public static final String BUILD_TYPE = "debug";
  public static final String FLAVOR = "";
  public static final int VERSION_CODE = 1;
  public static final String VERSION_NAME = "1.0";
}
```

BuildConfig类是Gradle构建脚本在编译后生的的，不能手动修改。里面储存了很多配置信息，java代码中可以直接引用，我们可以新增一些自定义的字段，来动态配置它们的值，gradle提供了`buildConfigField(String type, String name, String value)`函数让我们添加变量

```
productFlavors {
        google {
            buildConfigField 'String','WEB_URL','"http://www.google.com"'
        }
        baidu {
            buildConfigField 'String','WEB_URL','"http://www.baidu.com"'
        }
    }
```

这样BuildConfig中就会根据我们的渠道类型添加WEB_URL了。  
**value的值是单引号里面的东西，例如，String
类型的value，里面的双引号不能忽略，不然就报错了**

## 动态添加自定义资源

此处针对res/values类型的资源，可以在gradle中动态定义。使用`resValue`，可以在`·BuildType`、`ProductFlavor`两个对象中定义。  
如下：

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'
    }
}

apply plugin: 'com.android.application'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.1"

    defaultConfig {
        applicationId "org.flysnow.app.example97"
        minSdkVersion 14
        targetSdkVersion 23
        versionCode 1
        versionName '1.0.0'
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            zipAlignEnabled true
        }
    }
    productFlavors {
        google {
            resValue 'string','channel_tips','google渠道欢迎你'
        }
        baidu {
            resValue 'string','channel_tips','baidu渠道欢迎你'
            resValue 'color','main_bg','#567896'
            resValue 'integer-array','months','<item>1&lt/item>'
        }
    }
    aaptOptions {

    }
}
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```

## Java 编译选项

```
android{
	compileSdkVersion 23
	……
	
	compileOptions{
		encoding = 'utf-8'
		sourceCompatibility = JavaVersion.VERSION_1_6
		targetCompatibility = JavaVersion.VERSION_1_6
	}
}
```

compileOptions 是编译配置，有三个属性：encoding、sourceCompatibility、targetCompatibility。分别表示编码格式，Java源代码编译级别，生成的Java字节码的版本。后两个接收的值可以是以下几种格式：  
* `“1.6”` 
* `1.6` 
* `JavaVersion.VERSION_1_6`
* `VERSION_1_6`

## adb 操作选项配置

```
android{
	……
	adbOptions{
		timeOutInMs = 5*1000  //秒
		installOptions '-r', '-s'
	}
}
```

用的比较多的是timeOutInMs，设置执行adb这个命令的时间，有时候我们安装、运行或者调试程序的时候，可能会遇到 CommandRejectException 异常，这个一般是我们执行一个命令的时候，在规定的时间内没有返回应有的结果，这时我们可以把超时时间设长一点来解决。  
如果用到其他方法，可自行查阅adbOptions源码。

## dex选项配置

```
android{
	……
	
	dexOptions{
		incremental true
		javaMaxHeapSize '4g'
		jumboMode true
		threadCount 2
		preDexLibraries true
	}
}
```

具体作用自行查阅。

## 突破65535方法限制

Android 5.0以后版本，即将`Android Build Tools`和`Android Support Repository`升级到21.1及以上，这时支持Multidex功能的最低版本，然后配置

```
android{
	defaultConfig{
		multiDexEnabled true
	}
}
```

然后让我们的Application 继承MultiDexApplication，或者重写attachBaseContext（Context context）即可

```
public class myApp extends Application{
	
	@Override
	protected void attachBaseContext(Context context)
	{
		super.attachBaseContext(context);
		MultiDex.install(this);
	}
}
```

## 自动清理未使用的资源

```
shrinkResources true
```

会自动清理未使用的资源，但是无法识别反射等用到的资源，因此要配合keep一起，防止误删有用资源。  

在res/raw目录下建立keep.xml

```
<?xml version="1.0" encoding="utf-8"?>
<resources
    xmlns:tools="http://schemas.android.com/tools"
    tools:keep="@layout/getui_notification"/>
```


```
resConfig 'zh'
```

只打包配置过的资源，如上面，只打包zh资源（中文资源）