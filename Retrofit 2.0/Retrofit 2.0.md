# Retrofit 2.0

[链接] (https://www.jianshu.com/p/a3e162261ab6)

> 本文只是对Retrofit2.0做一些记录

## 目录

![] (image/rt1.png)

## 简介

![] (image/rt2.png)

* 准确的说，**Retrofit 是一个RESTFUL的HTTP网络请求框架的封装**，网络请求的工作本质上是OKHttp完成，而Retrofit仅负责网路请求接口的封装

![] (image/rt3.png)

* App通过Retrofit请求网路，实际上是使用Retrofit接口层封装请求参数、Header、Url等信息，之后由OKHttp完成后续请求工作；在服务端返回数据后，OKHttp将原始的结果交给Retrofit，Retrofit根据用户需求对结果进行解析。

## 与其他开源框架对比

![] (image/rt4.png)

## 注解

![] (image/rt5.png)


## 请求流程分析

![] (image/rt6.png)