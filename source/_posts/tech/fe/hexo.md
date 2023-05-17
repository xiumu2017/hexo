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
