---
title: Android开发-App启动流程
date: 2021-08-19 18:22:25
tags: Android
categories: Android
---

App的启动流程：

1. 点击桌面 App 图标，Launcher 进程采用 Binder IPC 向 system_server 进程发起 startActivity 请求；
2. system_server 进程接收到请求后，向 zygote 进程发起创建进程的请求；
3. Zygote 进程 fork 出新的子进程，即 App 进程；
4. App 进程，通过 Binder IPC 向 system_server 进程发起 attachApplication 请求；
5. system_server 进程在收到请求后，进行一系列准备工作后，再通过 binder IPC 向 App 进程发送 scheduleLaunchActivity 请求；
6. App 进程的 binder 线程 (ApplicationThread) 在收到请求后，通过 handler 向主线程发送 LAUNCH_ACTIVITY 消息；
7. 主线程在收到 Message 后，通过发射机制创建目标 Activity，并回调 Activity.onCreate() 等方法；
8. App 正式启动，开始进入 Activity 生命周期，执行完 onCreate/onStart/onResume 方法，UI 渲染结束后便可以看到 App 的主界面。
