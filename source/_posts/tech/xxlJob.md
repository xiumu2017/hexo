---
title: 分布式定时任务中间件 xxl-job 集成指南
excerpt: 分布式定时任务中间件 xxl-job SpringBoot 集成指南
date: 2022-1-7 18:33:30
index_img: https://paradise-1256237186.cos.ap-nanjing.myqcloud.com/Snipaste_2022-01-08_23-25-52.png
tags:
- SpringBoot
- 定时任务
- 分布式
categories:
- [Tech]
---

说明：基于 SpringBoot 2.5+

## 管理后台搭建

### MySQL 数据库初始化

新建并初始化数据库，sql 脚本地址：[https://gitee.com/xuxueli0323/xxl-job/blob/master/doc/db/tables_xxl_job.sql](https://gitee.com/xuxueli0323/xxl-job/blob/master/doc/db/tables_xxl_job.sql)

​

### 基于 docker-compose 搭建

docker-compose.yml

```yaml
version: '3'
services:
  gs-job:
    image: xuxueli/xxl-job-admin:${TAG}
    container_name: gs-job
    ports:
      - 8011:8080
    volumes:
      - /etc/localtime:/etc/localtime
      - ./log:/data/applogs
    environment:
      - PARAMS
      - TZ=Asia/Shanghai

networks:
  default:
    external: true
    name: gs-swarm_default

```

.env 环境变量文件中，配置数据库连接信息和镜像版本信息

```properties
TAG=2.3.0
PARAMS="--spring.datasource.url=jdbc:mysql://192.168.1.105:13306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai  --spring.datasource.username=root --spring.datasource.password=XnxDF1RnyxJuswcn"
```

启动：`docker-compose up -d`
​

### 管理平台页面

启动完成通过浏览器访问后台管理页面：[http://146.56.245.24:8011/xxl-job-admin](http://146.56.245.24:8011/xxl-job-admin)
默认的用户名和密码：admin/123456
​

**在执行器管理页面新增一个执行器，用于后续的执行器配置：**  
![image](https://image.zdzy.xyz/image/img20220106154021979.jpg)

​![image](https://image.zdzy.xyz/image/img20220106162958996.jpg)

这里的 AppName 用于后续执行器的配置。
​

## 执行器配置

### Step1：相关依赖引入

```xml
<!-- xxl-job-core -->
<dependency>
  <groupId>com.xuxueli</groupId>
  <artifactId>xxl-job-core</artifactId>
</dependency>
```

### Step2：增加配置

增加执行器配置文件，可以在 nacos 配置中心增加相关配置：

```yaml
xxl:
  job:
    admin:
      ### 调度中心部署跟地址 [选填]：如调度中心集群部署存在多个地址则用逗号分隔。执行器将会使用该地址进行"执行器心跳注册"和"任务结果回调"；为空则关闭自动注册；
      addresses: http://192.168.1.105:8011/xxl-job-admin
      ### 执行器通讯TOKEN [选填]：非空时启用；
      accessToken:
    executor:
      ### 执行器AppName [选填]：执行器心跳注册分组依据；为空则关闭自动注册
      appName: enterprise-admin
      ### 执行器注册 [选填]：优先使用该配置作为注册地址，为空时使用内嵌服务 ”IP:PORT“ 作为注册地址。从而更灵活的支持容器类型执行器动态IP和动态映射端口问题。
      address:
      ### 执行器IP [选填]：默认为空表示自动获取IP，多网卡时可手动设置指定IP，该IP不会绑定Host仅作为通讯实用；地址信息用于 "执行器注册" 和 "调度中心请求并触发任务"；
      ip:
      ### 执行器端口号 [选填]：小于等于0则自动获取；默认端口为9999，单机部署多个执行器时，注意要配置不同执行器端口；
      port: 9999
      ### 执行器运行日志文件存储磁盘路径 [选填] ：需要对该路径拥有读写权限；为空则使用默认路径；
      logPath: /data/gs-swarm/log/xxl-job
      ### 执行器日志文件保存天数 [选填] ： 过期日志自动清理, 限制值大于等于3时生效; 否则, 如-1, 关闭自动清理功能；
      logRetentionDays: 180
```

### Step3：执行器 Spring Bean 配置

配置文件映射类：

```java
package com.gaoshan.enterprise.admin.config;

import lombok.Data;
import lombok.ToString;
import org.springframework.boot.context.properties.ConfigurationProperties;

@Data
@ConfigurationProperties(prefix = "xxl.job")
public class XxlJobProperties {

    private Admin admin;
    private Executor executor;

    @Data
    @ToString
    static class Executor {
        private String appName;
        private String address;
        private String ip;
        private Integer port;
        private String logPath;
        private Integer logRetentionDays;
    }

    @Data
    @ToString
    static class Admin {
        private String addresses;
        private String accessToken;
    }
}
```

Spring Bean 声明：

```java
package com.gaoshan.enterprise.admin.config;

import com.xxl.job.core.executor.impl.XxlJobSpringExecutor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * xxl-job 执行器 bean 配置
 */
@Configuration
@Slf4j
@EnableConfigurationProperties(XxlJobProperties.class)
public class XxlJobExecutorConfig {

    private final XxlJobProperties properties;

    public XxlJobExecutorConfig(XxlJobProperties xxlJobProperties) {
        this.properties = xxlJobProperties;
    }

    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        log.info(">>>>>>>>>>> xxl-job config init.");
        log.info("properties : {} ", properties);
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(properties.getAdmin().getAddresses());
        xxlJobSpringExecutor.setAppname(properties.getExecutor().getAppName());
        xxlJobSpringExecutor.setIp(properties.getExecutor().getIp());
        xxlJobSpringExecutor.setPort(properties.getExecutor().getPort());
        xxlJobSpringExecutor.setAccessToken(properties.getAdmin().getAccessToken());
        xxlJobSpringExecutor.setLogPath(properties.getExecutor().getLogPath());
        xxlJobSpringExecutor.setLogRetentionDays(properties.getExecutor().getLogRetentionDays());
        return xxlJobSpringExecutor;
    }
}
```

### Step4：添加定时任务方法

这里使用了最简单的通过 `@XxlJob` **注解** 的方法：

```java
package com.gaoshan.enterprise.admin.scheduler;

import com.xxl.job.core.context.XxlJobHelper;
import com.xxl.job.core.handler.annotation.XxlJob;
import lombok.AllArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

/**
 * PdfGenScheduler
 *
 * @author Paradise
 * @date 2022/1/6 14:21
 **/
@Component
@AllArgsConstructor
@Slf4j
public class PdfGenScheduler {

    @XxlJob(value = "pdfGenJob")
    public void pdfGen() {
        XxlJobHelper.log(">>> start pdf gen job ...");
        log.info(">>> start pdf gen job ...");
    }
}

```

完成上述步骤就可以正常启动项目了。

### Step5：管理后台添加任务

![添加任务](https://image.zdzy.xyz/image/img20220106160426155.jpg#crop=0&crop=0&crop=1&crop=1&height=400&id=SuzvP&originHeight=953&originWidth=1323&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=555)

### Step6：启动任务，查看日志

启动任务：  
![启动](https://image.zdzy.xyz/image/img20220106160631258.jpg#crop=0&crop=0&crop=1&crop=1&height=433&id=ZlQnl&originHeight=542&originWidth=640&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=511)

任务正常调度执行：  
![日志](https://image.zdzy.xyz/image/img20220106161051457.jpg#crop=0&crop=0&crop=1&crop=1&id=LSPiY&originHeight=384&originWidth=1546&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## 参考文档

gitee 地址：[https://gitee.com/xuxueli0323/xxl-job](https://gitee.com/xuxueli0323/xxl-job)
官方文档：[https://www.xuxueli.com/xxl-job/](https://www.xuxueli.com/xxl-job/)
