---
title: Android开发-进程保活
date: 2021-08-25 14:03:44
tags: Android
categories: Android
---

> 这是一篇关于进程保活的总结文章

<!--more-->

#### 保活的两个方案

- 提高进程优先级，降低进程被杀死的概率
- 在进程被杀死后，进行拉活

#### 进程的优先级，划分五级：

1. 前台进程（Foreground process）
2. 可见进程（Visible process）
3. 服务进程（Service process）
4. 后台进程（Background process）
5. 空进程（Empty process）

##### **前台进程一般有以下特点：**

- 拥有用户正在交互的Activity（已调用onResume）
- 拥有某个Service，后者绑定到用户正在交互的Activity
- 拥有正在前台运行的Service（服务已调用startForeground）
- 拥有一个正执行生命周期回调的Service（onCreate，onStart或onDestory）
- 拥有其正执行其onReceive()方法的BroadcastReceiver

#### 提高进程优先级

- 利用Activity提升权限

  监控手机锁屏解锁事件，在屏幕锁屏时启动1像素的Activity，在用户解锁时将Activity销毁，注意该Activity需设计成用户无感知。

- Notification提升权限

  Android中Service的优先级为4，通过setForeground接口可以将后台Service设置为前台Service，使进程的优先级由4提升为2，从而是进程的优先级仅仅低于用户当前正在交互的进程，与可见进程优先级一致，使进程被杀死的概率大大降低。

#### 进程死后拉活

- 利用系统广播拉活

  在发生特定系统事件时，系统会发出相应的广播，通过在AndroidManifest中静态注册对应的广播监听器，即可在发生响应事件时拉活。

- 利用第三方应用广播拉活

  该方案总的设计思想与接收系统广播类似，不同的是该方案为接收第三方Top应用广播。通过反编译第三方Top应用，如微信、支付宝等等，找出它们外发的广播，在应用中进行监听，这样当这些应用发出广播时，就会将我们的应用拉活。

- 利用系统Service机制拉活

  将Service设置为START_STICKY，利用系统机制在Service挂掉后自动拉活。

- 利用Native进程拉活

  利用Linux中的fork机制创建Native进程，在Native进程中监控主进程的存活，当主进程挂掉后，在Native进程中立即对主进程进行拉活。

  原理：在Android中所有进程和系统组件的生命周期受ActivityManagerService的统一管理。而且，通过Linux的fork机制创建的进程为纯Linux进程，其生命周期不受Android管理。

- 利用 JobScheduler机制拉活

  在Android5.0以后系统对Native进程等加强了管理，Native拉活方式失效。系统在Android5.0以后版本提供了 JobScheduler接口，系统会定时调用该进程以使应用进行一些逻辑操作。

- 利用账号同步机制拉活

  Android系统的账号同步机制会定期同步账号进行，该方案目的在于利用同步机制进行进程的拉活。

 

