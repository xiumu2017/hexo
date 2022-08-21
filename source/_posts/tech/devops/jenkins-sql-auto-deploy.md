---
title: 基于 Jenkins 构建数据库自动部署方案
excerpt: 基于 Jenkins 构建数据库 SQL 脚本自动部署方案
date: 2022-6-27 14:35:07
index_img: https://paradise-1256237186.cos.ap-nanjing.myqcloud.com/941635-1653651727757wallhaven-28v8wm.jpg
tags:
- Devops
- Jenkins
- sql
categories:
- [Tech]
---

# 基于 Jenkins 构建数据库自动部署方案

## 整体思路

1. 通过 SVN 托管 数据库部署脚本，SQL 文件
2. Jenkins checkout 相关文件
3. Jenkins 流水线 Docker 客户端，alpine-linux 镜像，容器内安装 mysql-client
4. 通过 bash 命令聚合文件夹下的所有脚本文件，实现单条命令执行多个脚本文件的功能
5. 通过 mysql source 命令执行数据库部署脚本

## Pipeline Script

完整的 Pipeline Script如下:
```groovy
pipeline {
    agent any
    environment {
        HARBOR_CREDS = credentials('isolarerp-docker-creds')
        K8S_CONFIG = credentials('jenkins-k8s-config')
        SVN_CREDS_ID = "ops-svn"
    }
    parameters {
        string(name: 'SVN_URL', defaultValue: 'svn://192.168.188.109:32765/solareye/solareye-flyway/trunk/auto', description: '存放SQL脚本的SVN路径')
        string(name: 'DEPLOY_ENV', defaultValue: 'dev', description: '发布环境dev、fat、uat')
        string(name: 'DB_HOST', defaultValue: '192.168.188.109', description: '数据库Host')
        string(name: 'DB_USER', defaultValue: 'solareye', description: '数据库用户名')
        string(name: 'DB_PASS', defaultValue: '8c1VFREFGDRA@isolarerp', description: '数据库密码')
        string(name: 'DB_PORT', defaultValue: '30748', description: '数据库端口')
        string(name: 'DB_NAME', defaultValue: 'isolarerp', description: '数据库名称')
        string(name: 'SQL_EXAMPLE', defaultValue: 'select now()', description: '示例SQL')
        string(name: 'SQL_PATH', defaultValue: 'sql', description: 'SQL目录')
    }
    stages {
        stage('Checkout') {
            when {
                allOf {
                    expression { params.DEPLOY_ENV != 'pro' }
                }
            }
            agent any
            steps {
                checkout([$class: 'SubversionSCM',
                          additionalCredentials: [],
                          excludedCommitMessages: '',
                          excludedRegions: '',
                          excludedRevprop: '',
                          excludedUsers: '',
                          filterChangelog: false,
                          ignoreDirPropChanges: false,
                          includedRegions: '',
                          locations: [[credentialsId: SVN_CREDS_ID,
                                       depthOption: 'infinity',
                                       ignoreExternalsOption: true,
                                       local: '.',
                                       remote: params.SVN_URL
                                       ]],
                          workspaceUpdater: [$class: 'UpdateUpdater']])
            }
        }
        stage('SQL Runner') {
            when {
                allOf {
                    expression { params.DEPLOY_ENV != 'pro' }
                }
            }
            agent {
                docker {
                    image 'alpine:3.14'
                }
            }
            steps {
                sh "sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories"
                sh "apk add --no-cache mysql-client"
                sh "cd ${params.SQL_PATH} && ls |grep .sql |awk -F '\n' '{print \"source ./${params.SQL_PATH}/\"\$1}' > all.txt"
                sh "cat ${params.SQL_PATH}/all.txt"
                sh "mysql -h${params.DB_HOST} -u${params.DB_USER} -p${params.DB_PASS} -P${params.DB_PORT} -D${params.DB_NAME} -e '${params.SQL_EXAMPLE}'"
                sh "mysql -h${params.DB_HOST} -u${params.DB_USER} -p${params.DB_PASS} -P${params.DB_PORT} -D${params.DB_NAME} < ./${params.SQL_PATH}/all.txt"
            }
        }
    }
}
```

**注意：steps 中的 sh 命令只能使用双引号，单引号则无法获取构建参数；存在嵌套双引号时可以使用转义符 \  转义。**

## 使用方法

![jenkins-blue-ocean](https://image.zdzy.xyz/image/sungrow/img20220627143641636.jpg)

配置 SQL 脚本的 SVN 路径，数据库连接相关的信息，执行即可。


注意：目前仅支持执行配置的 SQL 目录下的数据库部署脚本，不支持递归或者多目录。

## Jenkins Pileline Agent 功能介绍

## Alpine-Linux 镜像介绍

Alpine 操作系统是一个面向安全的轻型 Linux 发行版。它不同于通常 Linux 发行版，Alpine 采用了 musl libc 和 busybox 以减小系统的体积和运行时资源消耗，
但功能上比 busybox 又完善的多，因此得到开源社区越来越多的青睐。在保持瘦身的同时，Alpine 还提供了自己的包管理工具 apk，
可以通过 https://pkgs.alpinelinux.org/packages 网站上查询包信息，也可以直接通过 apk 命令直接查询和安装各种软件。
Alpine 由非商业组织维护的，支持广泛场景的 Linux发行版，它特别为资深/重度Linux用户而优化，关注安全，性能和资源效能。
Alpine 镜像可以适用于更多常用场景，并且是一个优秀的可以适用于生产的基础系统/环境。
Alpine Docker 镜像也继承了 Alpine Linux 发行版的这些优势。
相比于其他 Docker 镜像，它的容量非常小，仅仅只有 5 MB 左右（对比 Ubuntu 系列镜像接近 200 MB），且拥有非常友好的包管理机制。官方镜像来自 docker-alpine 项目。
目前 Docker 官方已开始推荐使用 Alpine 替代之前的 Ubuntu 做为基础镜像环境。这样会带来多个好处。
包括镜像下载速度加快，镜像安全性提高，主机之间的切换更方便，占用更少磁盘空间等。

下表是官方镜像的大小比较：

```bash
REPOSITORY          TAG           IMAGE ID          VIRTUAL SIZE
alpine              latest        4e38e38c8ce0      4.799 MB
debian              latest        4d6ce913b130      84.98 MB
ubuntu              latest        b39b81afc8ca      188.3 MB
centos              latest        8efe422e6104      210 MB
```

## 待办事项

大数据量脚本执行效率验证

## 其它方案

1. SpringBoot ScriptUtils （缺点是和Java，SpringBoot 强绑定，优点是便于功能扩展开发）
2. Flyway 开源社区版（tidb兼容性不好，但是功能更为完善，比如支持版本控制，文件校验，多种使用方式等）
