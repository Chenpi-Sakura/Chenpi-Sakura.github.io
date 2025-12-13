---
title: 怎么解决无法拉取Docker镜像？不如我们自己建一个加速站（
autor: Chenpi
date: 2025-12-12 22:35:00 +0800
categories: [Docker]
tags: [Docker, 教程, ARC]

math: true
mermaid: true
---

> 该教程只能个人救急使用

最近在重新写 Jekyll 个人博客，然后要使用到 VSCode 的 DevContainer，在 Docker 容器里进行预览。

~~结果我手贱把以前的容器镜像都删了，第一步就卡死了。~~

我使用的主题[jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)的 `.devcontainer/devcontainer.json` 配置使用的是微软官方镜像：

```json
"image": "mcr.microsoft.com/devcontainers/jekyll:2-bullseye"
```

由于国内众所周知的网络原因，Docker Hub 和 MCR (Microsoft Container Registry) 基本处于断连状态。我在本地尝试重新构建镜像的时候反复报错 `Get "...": EOF` 或者 `net/http: TLS handshake timeout`，改了Docker Proxy 依然无效。

## 解决方案

我想了很久...找了很多加速的方法...然后...Gemini3帮我秒了...

>既然本地拉不下来，那就找个“海外中转站”！

核心思路：
1. 利用GitHub Actions拉取微软/官方镜像。
2. 在 Action 中把镜像推送到 阿里云容器镜像服务。
3. 本地 DevContainer 直接从阿里云拉取镜像。

~~不是GitHubActions还有阿里云容器镜像服务都是啥。~~

简单介绍一下：
- [GitHub Actions](https://github.com/features/actions)：GitHub 提供的“自动化流水线”。
    - **平时**：它在睡觉。
    - **触发**：当你提交代码或手动点击按钮时，它就被唤醒了。
    - **工作**：它会领一台全新的云端电脑（通常是 Ubuntu），照着你给的清单（.yml 文件）一行行执行命令。
    - **结果**：干完活（比如搬运镜像）后通知你，然后那台云电脑销毁。
- [阿里云容器镜像服务ACR](https://www.aliyun.com/product/acr)：阿里云提供的一个高性能、高可用、安全可靠的容器镜像托管平台。

而Github的服务器刚好就在国外，拉取这些镜像是很方便的，然后推送到阿里云，我们就能成功在国内拉取镜像了

## 操作步骤

### 第一步：开通阿里云容器镜像服务 (ACR)

1. 登录 [阿里云容器镜像服务控制台](https://cr.console.aliyun.com/)。
2. 创建个人实例（免费）。
3. 设置 Registry 登录密码：**注意这不是阿里云的登录密码，是专门给 Docker 用的固定密码。**
4. 创建命名空间（例如 `saku`）和镜像仓库（例如 `jekyll-mirror`）。
5. 可选：把仓库类型设置为**公开 (Public)**。这样本地拉取时不需要配置 Docker Login，非常方便。

### 第二步：配置 GitHub Actions 和 GitHub Secrets

首先fork这个项目[docker_sync](https://github.com/Chenpi-Sakura/docker_sync)，相关的`Action`代码可以参考项目`.github\workflows`目录下的yml文件

然后按照顺序点击 `Settings` -> `Secrets and variables` -> `Actions`->`New repository secret`

![](https://caipiischenpi.dpdns.org/PicGo/docker-sync-1.png)

>这里我们以拉取镜像`mcr.microsoft.com/devcontainers/jekyll:2-bullseye`和命名空间`saku`为例

需要设置以下参数

* `ALIYUN_USERNAME`: 你的阿里云 UserID。
* `ALIYUN_PASSWORD`: 第一步里设置的 Registry 独立密码。
* `ALIYUN_NAMESPACE`: 第一步中设置的命名空间
* `ALIYUN_REGISTRY`: 控制台概览中的公网域名

![](https://caipiischenpi.dpdns.org/PicGo/docker-sync-2.png)

填写后点击`Add secret`按照下图添加，添加以上四个Secret。

![](https://caipiischenpi.dpdns.org/PicGo/docker-sync-3.png)

配置完成后，去 GitHub 仓库的 `Actions` 页面，选中 `Mirror Docker Image to Aliyun`，点击 `Run workflow`，填写相关配置，再点击`Run workflow`。

![](https://caipiischenpi.dpdns.org/PicGo/docker-sync-4.png)

等待拉取完毕，

![](https://caipiischenpi.dpdns.org/PicGo/docker-sync-5.png)
![](https://caipiischenpi.dpdns.org/PicGo/docker-sync-6.png)

此时`Action`界面显示成功，控制台中也出现了相应的仓库

### 第四步：修改本地配置

回到本地项目，相应地修改 `.devcontainer/devcontainer.json` 中的image地址：

```json
"image": "crpi-xxxx.cn-hangzhou.personal.cr.aliyuncs.com/myblog/jekyll-mirror:latest"
```

> 地址位置参考仓库基本信息的公网地址

![](https://caipiischenpi.dpdns.org/PicGo/docker-sync-7.png)

此时重新拉取镜像，一下就成功了...

>如果拉取失败，可能是权限的问题，详细参考仓库信息中的操作指南

## 总结
通过这个方法，我们实际上是利用 GitHub Actions 搭建了一个 **“私人的 Docker 镜像加速通道”**。

这个方案需要自己手动配置一次 Workflow，且阿里云个人版有配额限制，但也算是无可奈何中的解决方案了吧。

祝大家 Coding 愉快！
