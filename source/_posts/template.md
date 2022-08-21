---
title: Template 模板
excerpt: 这是摘要，本文是编写博客的模板文件~
date: 2021/7/13 20:46:25
tags:
- 模板
- 测试
- Blog
categories:
- [Diary, PlayStation]
- [Diary, Games]
- [Life]
---

## H2 二级标题

### H3 三级标题

代码段：

```java
public class Object {
    private static native void registerNatives();

    @HotSpotIntrinsicCandidate
    public Object() {
    }

    @HotSpotIntrinsicCandidate
    public final native Class<?> getClass();

    @HotSpotIntrinsicCandidate
    public native int hashCode();

    public boolean equals(Object obj) {
        return this == obj;
    }
    public String toString() {
        return this.getClass().getName() + "@" + Integer.toHexString(this.hashCode());
    }

    @HotSpotIntrinsicCandidate
    public final native void notify();

    @HotSpotIntrinsicCandidate
    public final native void notifyAll();
        
}
```

来一段 HTML 试试

<div>
<b style="color: darkred">我是HTML</b>
</div>

图片功能：

![便签功能](https://image.zdzy.xyz/image/img20211111173449470.jpg)

{% note success %}
success tag test
{% endnote %}

<p class="note note-warning">Hello world ~</p>

更多插件使用功能 [更多插件](https://hexo.fluid-dev.com/docs/guide/#tag-%E6%8F%92%E4%BB%B6)
