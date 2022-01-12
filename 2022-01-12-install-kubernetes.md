---
title: 虚拟机配置 kubernetes 集群
date: 2022-01-12 21:00:00
description: 众所周知，kubernetes(文中简称k8s)已成为容器编排的首选工具，为了更好的使用和学习，但在安装过程中由于网络问题导致不顺利，该文主要记录安装过程中遇到的问题及解决方法。
categories:
- 服务器
tags:
- kubernetes
- k8s
---

# 环境配置

- VirtualBox 6.0
- Ubuntu 20.04(三台，一台做master，两台做node)
- Kubernetes 1.23(该版本当前最新稳定版) 

# 