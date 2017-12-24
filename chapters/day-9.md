大前端 Android 开发日记九：微博分享
===

早期在评估的时候，使用的是第三方 SDK —— shareSDK。可是，这个第三方 SDK 需要注册相应的账号，并且在后台，我们可以看到对应的分享数据。这也就意味着，它会收集一些用户信息，考虑到在这个因素，只好自己编写相应的对接逻辑。

因此，在这一天里，主要做的是微博的分享功能。

短信分享返回
---

在最近设计的时候，在完成短信分享之后，我需要知道它是通知短信分享出去的。就会用到 ``onActivityResult`` 来在 Activity 之间数据交流：

```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (requestCode == ShareUtil.SMS_REQUEST_CODE) {
        bottomSheetDialog.dismiss();
    }
}
```

这里的 requestCode 提供给 onActivityResult，是以便确认返回的数据是从哪个 Activity 返回的。这个 requestCode 和 startActivityForResult 中的 requestCode 相对应。即，如下的短信分享逻辑：

```
context.startActivityForResult(mIntent, ShareUtil.SMS_REQUEST_CODE);
```

微博分享接入
---

接下来的就是微博分享了，按照官方的文档接入 SDK。配置权限

```
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_SETTINGS"/>
```

对应的 filter

```
<intent-filter>
    <action android:name="com.sina.weibo.sdk.action.ACTION_SDK_REQ_ACTIVITY" />
    <action android:name="android.intent.action.MAIN"/>
</intent-filter>
```

在项目的 build.gradle 中添加依赖源

```
allprojects {
    repositories {
        jcenter()
        google()
        maven { url "https://dl.bintray.com/thelasterstar/maven/" }
    }
}
```

在应用的 build.gradle 添加对应的依赖

```
implementation 'com.sina.weibo.sdk:core:4.1.5:openDefaultRelease@aar'
```

然后创建一个实现 WbShareCallback 对象的 Activity：

```

@SuppressLint("Registered")
public class ShareableActivity implements WbShareCallback {
    private WbShareHandler shareHandler;
    private BottomSheetDialog bottomSheetDialog;
    private IWXAPI api;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        WbSdk.install(this, new AuthInfo(this, ShareUtil.weiboId, "http://www.phodal.com/", ""));

        shareHandler = new WbShareHandler(this);
        shareHandler.registerApp();
    }

    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        shareHandler.doResultIntent(intent, this);
    }

    @Override
    public void onWbShareSuccess() {
        bottomSheetDialog.dismiss();
    }

    @Override
    public void onWbShareCancel() {
        bottomSheetDialog.dismiss();
    }

    @Override
    public void onWbShareFail() {
        bottomSheetDialog.dismiss();
    }

    public void showShareDialog(InformationDetailModel information) {
        bottomSheetDialog = new BottomSheetDialog(ShareableActivity.this);
        View dialogView = LayoutInflater.from(ShareableActivity.this).inflate(R.layout.share_dialog, null);

        dialogView.findViewById(R.id.weibo_content).setOnClickListener(view -> {
            weiboShare(information);
        });

        bottomSheetDialog.setContentView(dialogView);
        bottomSheetDialog.show();
    }
    private void weiboShare(InformationDetailModel information) {
        WeiboMultiMessage weiboMessage = new WeiboMultiMessage();
        weiboMessage.mediaObject = ShareUtil.buildWebPageObject(this, information);
        shareHandler.shareMessage(weiboMessage, false);
    }
}
```

差不多就可以了。

