---
title: "Pod重启策略"
date: 2020-05-18T13:50:16+08:00
draft: false  
categories: [kubernetes]
tags: [kubernetes]

---

Pod 的重启策略

<!--more-->

Pod 在整个生命周期中被系统定义为各种状态，熟悉Pod的各种状态对于理解如何设置Pod调度策略。重启策略是有很有必要的

| 状态值    | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| Pending   | API server已经创建该Pod，但是Pod内还有一个或多个容器的镜像没有创建，包括正在下载镜像的过程 |
| Running   | Pod 内所有容器均已创建，且至少有一个容器处于运行状态、正在启动状态或正在重启状态 |
| Succeeded | Pod 内所有容器均成功执行后退出，且不会再重启                 |
| Failed    | Pod 内所有容器均已退出，但至少有一个容器退出为失败状态       |
| Unknown   | 由于某种原因无法获取该Pod的状态，可能由于网络通信不畅        |

Pod 的重启策略

Always: 当容器失效时，由kubelet自动重启该容器

Onfailure: 当容器终止运行且退出码不为0时，由kubelet 自动重启该容器

Never:  不论容器运行状态如何，kubelet 都不会重启该容器

kubelet 重启失效容器的时间间隔已sync-frequency 乘以2n 来计算，例如 1、2、4、8倍等，最长延时5分钟，并且在成功重启后10min后重置该时间



Pod的重启策略与控制方式息息相关，当前可用于管理Pod 的控制器包括ReplicationController、Job  DaemonSet 及直接通过kubelet 管理(静态Pod)，每种控制对Pod 的重启策略要求如下:

RC与DaemonSet ：必须设置为Always 

Job:  OnFailure 或Never，确保容器执行完成后不再重启

kubelet:  在Pod 失效时自动重启，不论将RestartPolicy 设置什么值，也不会对Pod 进行健康检查

