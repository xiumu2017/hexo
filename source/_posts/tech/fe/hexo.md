---
title: Hexo
excerpt: hexo 的原理解析，配置
date: 2023-5-17 15:09:59
tags:
- 前端
- TIL
categories:
- [Tech, FrontEnd]
---



## 本博客搭建指南

本地编辑工具：Typora

代码托管地址：[Github-Hexo](https://github.com/xiumu2017/hexo)

博客托管地址：[Vercel](https://vercel.com/xiumu2017s-projects/hexo)

域名解析：[腾讯云](https://blog.zdzy.xyz) 

图片服务：腾讯云 COS + CDN + Quicker 快捷方式

一开始因为设置了防盗链所以无法在 Typora 中预览图片，COS 桶安全管理中设置允许空 Referer 就好了。

![防盗链设置](https://txoss.zdzy.xyz/img/sungrow/20240514135704.png)



PicGo Win11 无法安装是什么情况？

准备重启下电脑试试，Still 不行

CDN加速：腾讯云 - 内容分发网络https://getquicker.net/Sharedaction?code=453910f7-b773-4a91-88f9-08dc5a32504a

![image](https://image.zdzy.xyz/image/sungrow/img20240514112013703.jpg)



## Hexo

### 主题 Theme

[Fluid 主题文档](https://hexo.fluid-dev.com/docs/start/#%E4%B8%BB%E9%A2%98%E7%AE%80%E4%BB%8B)

### 插件 Plugins

值得深入研究的几个插件：

1. [图片自动部署替换方案，CDN 加速](https://github.com/lxl80/hexo-deployer-cos-cdn)
2. [使用 ChatGPT 自动添加标签](https://github.com/declan-haojin/hexo-auto-tag) 

## Yarn

### Why Yarn

#### lock 文件

yarn.lock 文件是 Yarn 包管理工具自动生成的一个锁定文件，用于确保在安装项目依赖时，每个依赖包的版本都是固定的，以便于项目在不同环境中的稳定性和一致性。

yarn.lock 文件记录了项目依赖包的精确版本号，以及依赖包之间的依赖关系。它的作用是在使用 Yarn 命令安装依赖时，优先使用 yarn.lock 文件中记录的版本号，而不是从网络上下载最新版本，从而保证每次安装的依赖包版本都是一致的。

当多个开发者协同工作时，yarn.lock 文件可以避免因为不同环境下依赖包版本不一致导致的问题，保证项目在不同环境中的可重现性和稳定性。

### 本博客的自定义配置

```css
.text {
    font-family: Verdana, Arial, Helvetica, sans-serif
}
```
