大前端 Android 开发日记 14：纯 HTML 的 WebView Loading 效果
===

因为 HTTP 请求 + WebView 的渲染时间问题，我们决定为 WebView 添加一个 Loading 效果。于是，在这一天里，做的主要也就是这一方面的工作。

不过，一开始的时候，没有意识到我们的 WebView 和一般的 WebView 是不一样的，也就掉坑里了。我们的 WebView 是使用纯 HTML 渲染的文章内容，也就是后台只返回一个纯的 HTML 文本，不需要去远程请求。

通常的 WebView Loading 效果
---

一般来说，通过以下的代码就可以为 WebView 添加一个 Loading 显示和隐藏的效果：

```
public class AppWebViewClients extends WebViewClient {
    private ProgressBar progressBar;

    public AppWebViewClients(ProgressBar progressBar) {
        this.progressBar = progressBar;
    }

    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        // TODO Auto-generated method stub
        view.loadUrl(url);
        return true;
    }

    @Override
    public void onPageStarted(WebView view, String url, Bitmap favicon) {
        super.onPageStarted(view, url, favicon);
    }

    @Override
    public void onPageFinished(WebView view, String url) {
        // TODO Auto-generated method stub
        super.onPageFinished(view, url);
        progressBar.setVisibility(View.GONE);
    }
}
```

也就是在 ``onPageFinished`` 的时候，隐藏 progressBar。后来，我发现我们的 progressBar 不会显示，因为这里的 ``onPageFinished`` 是 WebView 完成 HTTP 请求触发的事件。


本地 WebView Loading 效果
---

于是，我改用覆写 WebView 的方式，通过 onProgressChanged 来实时监测变化，最后等到 100% 再隐藏 hide

```
public class ProgressWebView extends WebView {

    private AVLoadingIndicatorView spinner;

    public ProgressWebView(Context context, AttributeSet attrs) {
        super(context, attrs);

        setWebChromeClient(new WebChromeClient());
    }

    public void setSpinner(AVLoadingIndicatorView spinner) {
        this.spinner = spinner;
    }

    public class WebChromeClient extends android.webkit.WebChromeClient {
        @Override
        public void onProgressChanged(WebView view, int newProgress) {
            if (newProgress == 100) {
                spinner.hide();
            }
            super.onProgressChanged(view, newProgress);
        }
    }
}
``

对应的布局：


```
<com.wang.avi.AVLoadingIndicatorView
    android:id="@+id/spinner"
    app:indicatorColor="@color/cmb_gold"
    ...
    />

<com.cmb.plugin.information.view.ProgressWebView
    android:id="@+id/webview"
    ...
    />

```

尽管没有完全实现在 WebView 渲染完成时，再隐藏的方式，但是勉强可以完成需求。要在 WebView 渲染完时，再隐藏，最简单的方式就是流入 JavaScript，让其在 ``window.onload`` 时告诉原生代码，渲染已完成。

