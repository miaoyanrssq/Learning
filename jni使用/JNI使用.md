# JNI使用

> 在Android Studio 2.2以上版本中，创建项目的时候勾选C++选项即可自动完成jni的配置，可以在已有的文件中进行代码编写。
> 如果在已有的项目中添加jni的支持，需要以下步骤：
> 

## 1、创建cpp文件夹
在app/src/main/下创建cpp目录，并在此目录下添加cpp文件

## 2、创建CMakeLists.txt文件
在app/目录下创建CMakeLists.txt文件，文件，并在cmakeList.txt 文件添加编译描述。 

![] (image/jni1.png)

```
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.

add_library( # Sets the name of the library.
             native-lib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/native-lib.cpp )



# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.

find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries( # Specifies the target library.
                       native-lib

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

## 3、修改app的build.gradle
在build.gradle中增加对Cnake的支持

![] (image/jni2.png)

## 4、在java中调用

在java中加载本地so文件，创建native方法，就可以调用了，安装了LLDB的还可以对cpp文件进行debug。

```
package cn.zgy.jnitest2;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {

    static {
        System.loadLibrary("native-lib");
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        TextView hello = findViewById(R.id.hello);
        hello.setText(stringFromJNI());
    }


    public native String stringFromJNI();
}

```

以上就是jni的基本配置

## 新增c++ 文件

可以直接在cpp文件夹中新建cpp文件，然后在CMakeLists.txt中进行相关配置

![] (image/jni3.png)

![] (image/jni4.jpg)




[**一些注意事项**](https://blog.csdn.net/zeqiao/article/details/77893167)