大前端 Android 开发日记 10：微信分享
===

与之前的微博分享相比，微信分享就没有那么容易了——微信官方的 SDK 太差劲了。文章也写得像一坨屎——因为文档里的代码都是截图的。。。

微信分享 SDK 接入
---

按照官方的指南，添加对应的依赖

```
dependencies {
    compile 'com.tencent.mm.opensdk:wechat-sdk-android-with-mta:+'
}
```

对应的权限：

```
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

相关的混淆代码配置：

```
-keep class com.tencent.mm.opensdk.** { *; }
-keep class com.tencent.wxop.** { *; }
-keep class com.tencent.mm.sdk.** { *; }
```

注册 WXApi
---

首先在程序的 Applicaiton 模块中注册、初始化 WXApi，以方便我们在其它模块中使用：

```
public class InformationApplication extends Application {
    public static IWXAPI wxApi;

    @Override
    public void onCreate() {
        super.onCreate();

        ...

        registerWechat();
    }

    private void registerWechat() {
        wxApi = WXAPIFactory.createWXAPI(this, ShareUtil.weixinAppId, true);
        wxApi.registerApp(ShareUtil.weixinAppId);
    }
    ...
}
```

分享逻辑
---

IWXAPI 中有一个接口是判断用户是否安装微信，如果没有安装微信，我们应该提醒用户；如果安装了微信，那么才能分享。对应的代码如下所示：

```
private void wechatShareByType(Model model, int weixinShareType) {
    IWXAPI wxApi = Application.wxApi;

    boolean isWeixinShareable = wxApi.isWXAppInstalled() && wxApi.isWXAppSupportAPI();
    if (!isWeixinShareable) {
        notInstallWeixinToast();
        return;
    }
    SendMessageToWX.Req req = ShareUtil.buildWXWebPageObject(this, model, weixinShareType);
    wxApi.sendReq(req);
}
```

使用 WXEntryActivity 做出响应
---

WXEntryActivity 用于分享完后的响应，于是我们需要注册一个新的 WXEntryActivity，并且固定是在 **包名.wxapi** 目录下

```
<activity
    android:name=".wxapi.WXEntryActivity"
    android:exported="true" />
```

而这个 Activity 需要实现 IWXAPIEventHandler 的方法

```
public class WXEntryActivity extends AppCompatActivity implements IWXAPIEventHandler {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Application.wxApi.handleIntent(getIntent(), this);
    }

    @Override
    public void onResp(BaseResp resp) {
    }

    @Override
    public void onReq(BaseReq baseReq) {
    }
}
```

使用 EventBus 传递事件
---

为了响应上一步的事件，我使用了 EventBus 来做对应的处理。只需要在 WXEntryActivity 中的 Response 里 post 事件即可：

```
@Override
public void onResp(BaseResp resp) {
    WeixinEvent weiXin = new WeixinEvent(2, resp.errCode, "");
    EventBus.getDefault().post(weiXin);
    finish();
}
```

对应的在订阅者里，我需要先注册事件：

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ...
    EventBus.getDefault().register(this);
}

@Override
protected void onDestroy() {
    super.onDestroy();
    EventBus.getDefault().unregister(this);
}
```

然后才能使用：

```
@Subscribe
public void onEventMainThread(WeixinEvent weixinEvent) {

}
```

这样，我就可以才分享的 Activity 里获取分享的状态了。
