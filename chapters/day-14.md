大前端 Android 开发日记 14：纯 HTML 的 WebView Loading 效果
===

### 在线 WebView Loading 效果

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

###  本地 WebView Loading 效果

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


```
@SuppressWarnings("deprecation")
public class ProgressWebView extends WebView {

    private ProgressBar progressBar;
    private Handler handler;

    public ProgressWebView(Context context, AttributeSet attrs) {
        super(context, attrs);
        progressBar = new ProgressBar(context, null, android.R.attr.progressBarStyle);
        LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.WRAP_CONTENT);
        params.gravity= Gravity.CENTER;
        progressBar.setLayoutParams(params);

        handler = new Handler();

        addView(progressBar);
        setWebChromeClient(new WebChromeClient());
    }

    public void setProgressBar(ProgressBar progressBar) {
        this.progressBar = progressBar;
    }

    public class WebChromeClient extends android.webkit.WebChromeClient {
        @Override
        public void onProgressChanged(WebView view, int newProgress) {
            if (newProgress == 100) {
                progressBar.setProgress(100);
                handler.postDelayed(runnable, 200);
            } else {
                progressBar.setProgress(newProgress);
            }
            super.onProgressChanged(view, newProgress);
        }
    }

    private Runnable runnable = new Runnable() {
        @Override
        public void run() {
            progressBar.setVisibility(View.GONE);
        }
    };
}
``



