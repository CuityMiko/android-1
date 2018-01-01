大前端 Android 开发日记 14：Android WebView 默认 margin 样式问题
===


```
public class WebViewUtil {
    public static String removeDefaultStyle(String originContent) {
        return "<body style='margin:0;padding:0;background-color: transparent;'>" + originContent + "</body>";
    }
}
```
