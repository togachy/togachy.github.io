---
layout: post
title: "Android卡顿检测"
subtitle: "Android卡顿相关知识"
date: 2022-02-16
author: "Togachy"
header-img: "img/post-bg.jpeg"
tags: 
    - Andorid
---


最近的一周的工作由于一些业务安排进行了调整，于是做一个阶段性的总结。 

抛开业务的部分，只谈技术。近期尝试了一下，监控Andoird APP的线上用户卡顿情况。

分析了新旧几款开源和商业化的组件，matrix，ArgusAPM, UMeng, Weibo, DoraemonKit。

虽然每个APM性能监控工具的文档写的都不一样，支持的功能，日志上报机制，后期的数据分析，问题归类，纬度和指标不一样。

但是从最初的监控层面看，大家的方案其实是一致的。


> P.S.  针对加密商业代码，不具体描写详细的代码。

从一个精简的文章看，其实核心类归类为一个。

```java
    Looper.getMainLooper().setMessageLogging(new Printer() {
                @Override
                public void println(String x) {
                    根据String参数进行判断
                    //println(">>>>> Dispatching to " ...);
                    UI线程执行开始，通过>>>>>, 或者Dispatching
                    //println("<<<<< Finished to " ...);
                    UI线程执行结束，通过<<<<<, 或者Finished

                    /**
                     * 通过开始和结束时间点
                     * 判断是否出现卡顿
                     */
                    performanceHook();
                }
            });

```


其他的功能，诸如，日志的持久化，上报时机等，还需要进一步细化开发。

此外

在这个监控的逻辑基础上。

还会涉及如下几个问题。

* 启动卡顿 Application启动过程， attachBaseContext，onCreate等生命周期，都没有处理。
* UI卡顿过程中，Looper没有执行到"<<<<< Finished to "用户主动关闭APP，或者在绘制过程中APP崩溃了。 当前这次卡顿的统计和记录。
* 非UI线程导致的CPU满载，IO流阻塞等资源不足情况下，设备卡顿。

针对这几个问题

对应的项目也基本通过启动时间的监控

CPU，内存信息等数据的监控，进行了分析和问题定位。

像DoraemonKit也提供了gradle的plugin对APP的打包过程，进行子节码插桩，用于定位异步线程的执行情况。

基本就完善了性能监控组件，对APP卡顿的完整分析。

目前这两天的技术调研就截止到此处了。


---

2022-2-18

此外，近期又看到了一个新的性能监控工具Mars-APMPlus

更加丰富了以开发者作为业务使用者的更完善的场景。

引入Desktop软件 （其实我个人本来感觉这种自己开发的功能，肯定会被AS，google develop推出的profile插件完爆）

但是基于不同的思路，Mars这个工具，通过结合机器学习，强化学习，UI自动化测试，从另一个角度完善了性能监控的能力。

感觉很厉害，后续会仔细看看，再更新学习情况。


保存下链接<a href="https://my.oschina.net/u/4180867/blog/5396533">Mars-APMPLus</a>



