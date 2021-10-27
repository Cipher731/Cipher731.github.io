---
title: Docker源码分析 - 1 - docker create
tags:
- Docker
- Cloud Native
- Code Audit
---

## 0. 系列前言
Docker源码分析是一个新挖的坑，分析Docker的源码肯定是研究容器安全绕不过去的一道坎，肯定要知道容器运行的整个流程背后的逻辑是怎么养的，不能一直停留在配置层面，那其实根本无法接触到核心原理。

我的目标是完全分析moby、containerd和runc的所有API，但是大概率又立了一个flag，xD

## 1.
