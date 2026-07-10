---
title: ✨K8S 基础操作指南
published: 2024-10-22
pinned: false
description: "整理 Kubernetes 常用基础操作命令，包含 Secret、ConfigMap、Service、Deployment 等资源的快速部署示例。"
tags: [K8S, Linux]
category: 技术分享
draft: false
image: ../uploads/20260110/mt.png
---

# K8S 基础操作指南

### **快速部署命令**

```bash
# 1. 创建 Secret 配置
kubectl apply -f sercet-test.yaml -n <namespace>

# 2. 创建 ConfigMap 配置
kubectl apply -f cm-test.yaml -n <namespace>

# 3. 创建 Service 网络服务
kubectl apply -f svc-test.yaml -n <namespace>

# 4. 创建 Deployment 部署
kubectl apply -f deploy-test.yaml -n <namespace>

```

### **常用查看命令**

```bash
# 查看所有 Pod
kubectl get pods -n <namespace>

# 实时监控 Pod 状态
kubectl get pods -n <namespace> -w

# 查看指定 Pod 详细信息
kubectl describe pod <pod-name> -n <namespace>

# 查看所有服务
kubectl get services -n <namespace>

```

### **日志查看命令**

```bash
# 查看实时日志
kubectl logs -f <pod-name> -n <namespace>

# 查看最近的日志(最后100行)
kubectl logs --tail=100 <pod-name> -n <namespace>

# 查看之前崩溃容器的日志
kubectl logs <pod-name> -n <namespace> --previous

```

### **容器操作命令**

```bash
# 进入容器 Shell
kubectl exec -it <pod-name> -n <namespace> -- /bin/bash

# 在容器中执行单个命令
kubectl exec <pod-name> -n <namespace> -- <command>

```

### **删除操作命令**

```bash
# 删除 Deployment
kubectl delete deployment business -n <namespace>

# 删除 Service
kubectl delete service <service-name> -n <namespace>

# 删除 Pod(通常会自动重建)
kubectl delete pod <pod-name> -n <namespace>

```

---

## **K8S 命令参考手册**

> kubectl 控制 Kubernetes 集群管理器 更多信息请访问: [https://kubernetes.io/docs/reference/kubectl/](https://kubernetes.io/docs/reference/kubectl/)

### **基本命令（初学者）**


| **命令** | **说明**                                                  |
| -------- | --------------------------------------------------------- |
| `create` | 从文件或标准输入创建资源                                  |
| `expose` | 将复制控制器、服务、部署或 Pod 暴露为新的 Kubernetes 服务 |
| `run`    | 在集群上运行特定镜像                                      |
| `set`    | 为对象设置指定特性                                        |

### **基本命令（中级）**


| **命令**  | **说明**                                                       |
| --------- | -------------------------------------------------------------- |
| `explain` | 获取资源的文档说明                                             |
| `get`     | 显示一个或多个资源                                             |
| `edit`    | 编辑服务器上的资源                                             |
| `delete`  | 通过文件名、标准输入、资源和名称或通过资源和标签选择器删除资源 |

### **部署命令**


| **命令**    | **说明**                                   |
| ----------- | ------------------------------------------ |
| `rollout`   | 管理资源的部署                             |
| `scale`     | 为部署、副本集或复制控制器设置新大小       |
| `autoscale` | 自动扩缩部署、副本集、有状态集或复制控制器 |

### **集群管理命令**


| **命令**       | **说明**                     |
| -------------- | ---------------------------- |
| `certificate`  | 修改证书资源                 |
| `cluster-info` | 显示集群信息                 |
| `top`          | 显示资源（CPU/内存）使用情况 |
| `cordon`       | 标记节点为不可调度           |
| `uncordon`     | 标记节点为可调度             |
| `drain`        | 清空节点以准备维护           |
| `taint`        | 更新一个或多个节点上的污点   |

### **故障排查和调试命令**


| **命令**       | **说明**                                 |
| -------------- | ---------------------------------------- |
| `describe`     | 显示特定资源或资源组的详细信息           |
| `logs`         | 打印 Pod 中容器的日志                    |
| `attach`       | 挂接到一个运行中的容器                   |
| `exec`         | 在某个容器中执行一个命令                 |
| `port-forward` | 将一个或多个本地端口转发到某个 Pod       |
| `proxy`        | 运行一个指向 Kubernetes API 服务器的代理 |
| `cp`           | 从容器复制文件和目录或复制到容器         |
| `auth`         | 检查授权                                 |
| `debug`        | 创建调试会话以排查工作负载和节点问题     |

### **高级命令**


| **命令**    | **说明**                                 |
| ----------- | ---------------------------------------- |
| `diff`      | 比较实时版本与将要应用的版本的差异       |
| `apply`     | 通过文件名或标准输入将配置应用到资源     |
| `patch`     | 更新资源的字段                           |
| `replace`   | 通过文件名或标准输入替换资源             |
| `wait`      | 实验性功能：等待一个或多个资源的特定条件 |
| `kustomize` | 从目录或 URL 构建 kustomization 目标     |

### **设置命令**


| **命令**     | **说明**                                                           |
| ------------ | ------------------------------------------------------------------ |
| `label`      | 更新某资源上的标签                                                 |
| `annotate`   | 更新一个资源的注解                                                 |
| `completion` | 为指定的 shell（bash、zsh、fish 或 powershell）输出 shell 补全代码 |

### **其他命令**


| **命令**        | **说明**                                           |
| --------------- | -------------------------------------------------- |
| `alpha`         | alpha 阶段功能的命令                               |
| `api-resources` | 打印服务器上支持的 API 资源                        |
| `api-versions`  | 以 "group/version" 形式打印服务器上支持的 API 版本 |
| `config`        | 修改 kubeconfig 文件                               |
| `plugin`        | 提供与插件交互的实用工具                           |
| `version`       | 输出客户端和服务端的版本信息                       |

### **使用说明**

```bash
kubectl [flags] [options]

# 获取命令帮助
kubectl <command> --help

# 查看全局选项
kubectl options
```
