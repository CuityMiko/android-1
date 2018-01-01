大前端 Android 开发日记 14：Android WebView 默认 margin 样式问题
===

这一也没啥做的，不过也算是修了一个 CSS 的样式问题。

这个问题的主要来源是，后端返回的 HTML 是不带样式的，而默认的 WebView 会为 Div 标签加上 8px 的 padding，也就是说没有 CSS Reset。这个时候，就需要手动加一个 Reset 效果了，于是就写了一个简单的方法来做这件事。顺便带页面的内容，包在了 body 中。

```
public class WebViewUtil {
    public static String removeDefaultStyle(String originContent) {
        return "<body style='margin:0;padding:0;background-color: transparent;'>" + originContent + "</body>";
    }
}
```

Reset 的 CSS 的内容就是：

```
margin:0;
padding:0;
```

然后在一次新的测试中，发现了样式中又默认了白色背景，于是就加了一个背景透明 ``background-color: transparent;``。

