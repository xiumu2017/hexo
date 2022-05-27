---
title: 什么是 SSE （Server-Sent Events）?
excerpt: 本文介绍 SSE 的基础概念，以及基于 SpringBoot 的简单实现
date: 2022-5-27 19:33:28
index_img: https://paradise-1256237186.cos.ap-nanjing.myqcloud.com/941635-1653651727757wallhaven-28v8wm.jpg
tags:
- SpringBoot
- SSE
- 消息推送
- websocket
categories:
- [Tech]
---

### 什么是 SSE （Server-Sent Events）?

严格地说，HTTP 协议无法做到服务器主动推送信息。但是，有一种变通方法，就是服务器向客户端声明，接下来要发送的是流信息（streaming）。
也就是说，发送的不是一次性的数据包，而是一个数据流，会连续不断地发送过来。这时，客户端不会关闭连接，会一直等着服务器发过来的新的数据流，视频播放就是这样的例子。本质上，这种通信就是以流信息的方式，完成一次用时很长的下载。

SSE 就是利用这种机制，使用流信息向浏览器推送信息。它基于 HTTP 协议，目前除了 IE/Edge，其他浏览器都支持。

### SSE 对比 Websocket 的优点？

提到服务端数据推送，你可以一下子就想到了Websocket，WebSocket是一种全新的协议，随着HTML5草案的不断完善，越来越多的现代浏览器开始全面支持WebSocket技术了，它将TCP的Socket（套接字）应用在了webpage上，从而使通信双方建立起一个保持在活动状态连接通道。
WebSocket 和 SSE 都是传统请求-响应 Web 架构的替代方案，但它们不是完全冲突的技术。

WebSocket 架构在客户端与服务器之间打开一个套接字，用于实现全双工（双向）通信。 无需发送 GET 消息并等待服务器响应，客户端只需监听该套接字，接收服务器更新，并使用收到的数据来发起或支持各种交互。 客户端也可以使用套接字与服务器通信，例如在成功收到更新时发送 ACK 消息。

SSE 是一种更简单的标准，是作为 HTML5 的扩展而开发的。 尽管 SSE 支持从服务器向客户端发送异步消息，但客户端无法向服务器发送消息。 对于客户端只需接收从服务器传入的更新的应用程序，SSE 的半双工通信模型最适合。 与 WebSocket 相比，SSE 的一个优势是它是基于 HTTP 而运行的，不需要其他组件。

总结：
1. SSE 使用 HTTP 协议，现有的服务器软件都支持。WebSocket 是一个独立协议。
2. SSE 属于轻量级，使用简单；WebSocket 协议相对复杂。
3. SSE 默认支持断线重连，WebSocket 需要自己实现。
4. SSE 一般只用来传送文本，二进制数据需要编码后传送，WebSocket 默认支持传送二进制数据。
5. SSE 支持自定义发送的消息类型。

### 客户端示例代码

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>SSE Demo</title>
</head>
<body>
    <div id="page"></div>
    <script>
        // SSE的API在EventSource对象上
        // 可以使用 if('EventSource' in window) 判断浏览器是否支持SSE
        // 建立SSE连接，直接如下创建EventSource实例，支持跨越
        var source = new EventSource("http://127.0.0.1:8080/message/sse/subscribe?userId=paradise");
        // EventSource.readyState代表连接状态，有以下三种情况
        // 0 —> 连接还未建立，或者正在断线重连
        // 1 -> 连接已建立，可以接受数据
        // 2 -> 连接已关闭或请求错误
        var div = document.getElementById("page");
        // 连接创建成功的回调事件
        source.onopen = function (event) {
            div.innerHTML += "<p>Connection open ...</p>";
        };
        // 连接创建失败的回调事件
        source.onerror = function (event) {
            div.innerHTML += "<p>Connection close.</p>";
        };
        // 自定义事件，服务端返回时设置event字段为自定义事件名称
        source.addEventListener("connecttime",
            function (event) {
                div.innerHTML += "<p>Start time: " + event.data + "</p>";
            },
            false
        );
        // 接受到数据的回调事件，event未特殊设置时，默认是message
        source.onmessage = function (event) {
            div.innerHTML += "<p>message event: " + event.data + "</p>";
        };
        // 关闭连接  source.close();
    </script>
</body>
</html>
```

### 服务端示例代码

基于 SpringBoot：

接口定义，重点是 produces = MediaType.TEXT_EVENT_STREAM_VALUE 

``` java
    /**
     * SSE 消息推送订阅
     *
     * @param userId 当前用户ID，理论上应该通过 token 获取
     * @return {@link SseEmitter}
     */
    @ApiOperation("SSE 服务端消息推送订阅")
    @GetMapping(path = "/subscribe", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    SseEmitter subscribe(@RequestParam String userId) {
        log.info("SSE 新用户加入：userId : {} ", userId);
        return sseMessageComponent.join(userId);
    }
```

新用户加入的方法实现：

```java
    /**
     * 新用户加入
     *
     * @param userId 当前用户ID
     * @return {@link SseEmitter}
     */
    public SseEmitter join(String userId) {
        SseEmitter sseEmitter = emitterMap.get(userId);
        if (sseEmitter == null) {
            sseEmitter = new SseEmitter(-1L);
            sseEmitter.onCompletion(() -> {
                log.info("SSE 用户退出：userId : {} ", userId);
                emitterMap.remove(userId);
            });
            sseEmitter.onError(throwable -> {
                log.info("SSE 用户Error：userId : {} ", userId);
                log.error("SSE error : {} ", throwable.getLocalizedMessage());
                emitterMap.remove(userId);
            });
            sseEmitter.onTimeout(() -> {
                log.info("SSE 用户超时：userId : {} ", userId);
                emitterMap.remove(userId);
            });
            emitterMap.put(userId, sseEmitter);
        }
        this.sendMsg(userId, "Welcome to Solareye SSE");
        return sseEmitter;
    }
```

消息推送的方法实现：
 ```java
    /**
     * 消息发送
     *
     * @param userId 用户ID
     * @param msg    消息内容
     */
    public void sendMsg(String userId, String msg) {
        SseEmitter sseEmitter = emitterMap.get(userId);
        if (sseEmitter != null) {
            try {
                sseEmitter.send(msg);
            } catch (IOException e) {
                log.error("消息内容：msg : {} ", msg);
                log.error("SSE 消息发送失败：{}", e.getLocalizedMessage(), e);
            }
        } else {
            log.error("消息内容：msg : {} ", msg);
            log.error("用户【userId：{}】 已离线，SSE 消息发送失败 ", userId);
        }
    }
```

完整的 Java 后端代码实现可参考 solareye-message 项目
参考文档: <https://www.ruanyifeng.com/blog/2017/05/server-sent_events.html>
服务端示例代码：<https://github.com/aliakh/demo-spring-sse>
SSE  浏览器 API 和协议（英文）：<https://hpbn.co/server-sent-events-sse/>

### 潜在 or 待解决问题

SSE 连接对后端服务资源消耗占用情况？
如何评估单位机器可承载的最大连接数量？
基于 LRU 实现 SSE 连接淘汰策略
