#Android Gradle 记录(1)

> 从今天开始记录一下Android gradle的知识


##Android Gridle插件分类

* App插件 id： com.android.application
* Library插件 id: com.android.library
* Test插件 id: com.android.test

通过应用以上3种不同的插件，就可以配置app工程、library工程、test测试工程。

##应用Android Gradle插件
要应用一个插件，必须知道他们的插件id，除此之外，如果是第三方插件，还要配置依赖的classpath，Android Gridle插件属于第三方插件，它托管在Jcenter上，所以在应用它们之前，我们要先配置依赖的classpath，这样我们应用插件的时候，Gradle系统才能找到它们。

```
buildscript{
	repositories{
		jcenter()
	}
	dependencies{
		classpath 'com.android.tools.build:gradle:2.3.3'
	}
}
```

我们配置仓库为jcenter， 这样当我们配置依赖的时候，Gradle就会去这个仓库里寻找我们的依赖。  
然后我们在dependencies{}里配置，我们需要的是Android Gridle2.3.3版本的插件。

buildscript{} 这部分的配置可以写到根工程的build.gradle脚本文件中，这样所有的子工程就不用重复配置了。以上配置好后，我们就可以应用我们的Android Gridle插件了：

```
apply plugin:'com.android.application'

android{
	compileSdkVersion 23
	buildToolsVersion "23.0.1"
	……
}
```

## 工程示例

在Android studio中新建一个app，什么都不要改动，直接查看根目录的build.gradle,以及app目录下的build.gradle文件

根目录build.gradle文件：

```
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

```

app目录下的build.gradle文件：

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 26
    buildToolsVersion "27.0.1"
    defaultConfig {
        applicationId "com.example.gaoyangzhen.autoview"
        minSdkVersion 15
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile project(':autoviewlibrary')
    compile 'com.android.support:appcompat-v7:26.+'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    testCompile 'junit:junit:4.12'
}

```

## 要点介绍

本文只对一些需要详细说明或者默认插件里面没有的知识做介绍，基础知识请自行查阅。

### 配置签名信息

```
Android {
	compileSdkVersion 23
	……
	
	signingConfigs{
		release{
			storeFile file("releasekey.keystore")
			storePassword "password"
			keyAlias "releaseKey"
			keyPassword "password"
		}
		debug{
			storeFile file("debugkey.keystore")
			storePassword "password"
			keyAlias "debugKey"
			keyPassword "password"
		}
	}
	……
}
```

debug模式有默认签名，如不需要，可以不配置。

签名的引用：

```
buildTypes{
	release{
		signingConfig signingConfigs.release
	}
	debug{
		signingConfig signingConfigs.debug
	}
}

或者直接在defaultConfig中：

defaultConfig{
	signingConfig signingConfigs.debug
}

```
如果你还有其他类型，想要为其配置单独的签名，也可以这样做，如特殊渠道包，付费vip包等。

### 构建的应用类型（buildTypes）

Android Gridle已经帮我们内置了debug和release两个构建类型，它们的区别主要是能否在设备上调试以及签名不一样，其他代码和文件资源都一样。如果想增加新的构建类型，在buildTypes{}代码快中继续添加就可以。此处列举一些常用配置：

#### applicationSuffix
applicationSuffix是BuildType的一个属性，用于配置基于默认applicationId的后缀，如默认applicaionId为com.zgy.example，我们在debug的BuildType中指定applicationSuffix为.debug,那么构建生成的debug apk的报名就是com.zgy.example.debug

#### debuggable
用于配置是否生成一个可供调试的apk，值为true或false

#### jniDebuggable
配置是否生成一个可供调试Jni代码的apk，（true，false）

#### minifyEnabled
配置该BuildType是否启用Proguard混淆，（true，false）

#### multiDexEnabled
配置该BuildType是否启用自动拆分多个Dex的功能（true，false）

#### proguardFile
配置Progauard混淆使用的配置文件

#### proguardFiles
配置多个混淆文件

#### shrinkResources
配置是否自动清理未使用的资源，默认false

#### signingConfig
配置签名

每个BuildType都会生成一个SourceSet，默认位置为src/。一个SourceSet包含源代码、资源文件以及AndroidManifest文件等，所以，针对不同的的BuildType，我们可以单独为其指定Java源代码、res资源等，之后把他们放在src/下相应的位置即可，在构建的时候，Android Gradle会优先使用它们代替main下的相关文件。  
新增BuildType的名字不能重复，并且不能占用系统的main和AndroidTest为名字。  
每个BuildType还会生成相应的assemble任务，执行相应的assemble任务，就能想成对应的apk。

### 启用zipalign优化
zipalign是一个整理优化aok文件的工具，能提高系统和应用的运行效率，更快的读写apk中的资源，降低内存的使用，所以，发布app前，一定要使用aipalign进行优化：

```
buildTypes{
	release{
		zipAlignEnabled true
	}
}
```