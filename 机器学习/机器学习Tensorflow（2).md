#机器学习Tensorflow（2）-安装

## 什么事Tensorflow
TensorFlow是Google开发的一款神经网络的Python外部的结构包, 也是一个采用数据流图来进行数值计算的开源软件库.TensorFlow 让我们可以先绘制计算结构图, 也可以称是一系列可人机交互的计算操作, 然后把编辑好的Python文件 转换成 更高效的C++, 并在后端进行计算.

## 为什么要使用Tensorflow
TensorFlow 无可厚非地能被认定为 神经网络中最好用的库之一. 它擅长的任务就是训练深度神经网络.通过使用TensorFlow我们就可以快速的入门神经网络, 大大降低了深度学习（也就是深度神经网络）的开发成本和开发难度. TensorFlow 的开源性, 让所有人都能使用并且维护, 巩固它. 使它能迅速更新, 提升.

## Tensorflow安装

安装 Tensorflow 时需要注意的几点:

1、MacOS, Linux, Windows 系统均已支持 Tensorflow。  
2、确定你的 python 版本  
3、你的 GPU 是 NVIDIA, 就可以安装 GPU 版本的 Tensorflow; 你的 GPU 不是 NVIDIA 也没有关系, 安装 CPU 版本的就好了.

###Linux和MacOS

本文将提到第一种最简单的安装方式, pip 安装. 使用 pip 安装的时候要确保你的 pip 已经存在于你的电脑中. 如果还没有安装 pip. 你可以在 Terminal 窗口中运行这个, 升级必要的组件:

```
# Ubuntu/Linux 64-位 系统的执行代码:
$ sudo apt-get install python-pip python-dev

# Mac OS X 系统的执行代码:
$ sudo easy_install --upgrade pip
$ sudo easy_install --upgrade six
```

####CPU版

Tensorflow (0.12之后) 做了更新, 绕过了复杂的安装步骤, 如果你只需要安装 CPU 版本的 Tensorflow, 运行下面这个就好了:

```
# python 2+ 的用户:
$ pip install tensorflow

# python 3+ 的用户:
$ pip3 install tensorflow
```

注意: 你需要8.1或更高版的 pip 才能顺利安装.

#### GPU版

Tensorflow 已经不再支持 mac 的 GPU 版了, 下面是 Linux 安装 GPU 版的说明. 说先安装 NVIDIA CUDA 必要组建.

```
$ sudo apt-get install libcupti-dev
```
然后确保你的 linux 上 pip 是可用的, 接着我们可以直接通过pip 安装:

```
$ sudo apt-get install python-pip python-dev   # for Python 2.7
$ sudo apt-get install python3-pip python3-dev # for Python 3.n
```
然后选择你想要cpu 或者 gpu 版本.

```
$ pip install tensorflow      # Python 2.7; CPU support (no GPU support)
$ pip3 install tensorflow     # Python 3.n; CPU support (no GPU support)
$ pip install tensorflow-gpu  # Python 2.7;  GPU support
$ pip3 install tensorflow-gpu # Python 3.n; GPU support
```

###Windows

安装前的检查:

* 目前只支持 Python 3.5/3.6 (64bit) 版本
* 你有安装 numpy (没有的话,请看这里numpy 安装教程)


接下来惊心动魄啦! 在 command 窗口中执行

```
# CPU 版的
C:\> pip3 install --upgrade tensorflow

# GPU 版的
C:\> pip3 install --upgrade tensorflow-gpu
```
**注意**

Windows 运行 Tensorflow 如果遇到这个报错:

```
Error importing tensorflow.  Unless you are using bazel,
you should not try to import tensorflow from its source directory;
please exit the tensorflow source tree, and relaunch your python interpreter
from there.
```
不要惊慌, 尝试下载安装 Windows 的 Microsoft Visual C++ 2015 redistributable update 3 64 bit. 就能解决这个问题.

* 或者在 Windows 运行的时候出现了如下报错, 你需要安装 Windows 的 Visual C++ Redistributable for Visual Studio 2015 就能成功解决问题.

```
ImportError: No module named '_pywrap_tensorflow_internal'
```

###测试 

然后打开你的 python 编辑器, 输入

```
import tensorflow
```
运行脚本来检查一下是否有正确安装.

###更新 Tensorflow 

最后, 如果你需要升级 Tensorflow 的版本, 推荐的方式是:

根据你的 python 版本, 在 terminal 中删除原有的版本

```
# 如果你是 Python 2, 请复制下面
pip uninstall tensorflow

# 如果你是 Python 3, 请复制下面
pip3 uninstall tensorflow
```
然后重复这个安装教程的步骤, 从头安装新版本.