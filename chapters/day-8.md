大前端 Android 开发日记八：Android 短信、微信、微博分享
===

在纠结了几天的图表功能之后，我开始开发一个新的功能。即分享内容到短信、微信、微博等渠道，对应的我有一个简单的 Task：

 - 在 Toolbar 写分享的按钮
 - 绘制一个 Android 的分享页面
 - 编写短信分享示例
 - 编写社交分享

在这一天，我只完成了前面的三部分。

Toolbar 上的分享按钮
---

在 Toolbar 主要还是靠 ImageView 来绘制右上角的分享按钮：

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.Toolbar xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:toolbar="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    android:background="?attr/colorPrimaryDark"
    android:gravity="center">

    <TextView
        android:id="@+id/toolbar_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="xxx" />

    <ImageView
        android:visibility="invisible"
        android:id="@+id/share"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingEnd="@dimen/length_24"
        android:paddingStart="@dimen/length_16"
        android:paddingTop="@dimen/length_16"
        android:paddingBottom="@dimen/length_16"
        android:layout_gravity="right"
        android:src="@drawable/share_icon"
        tools:ignore="RtlHardcoded" />

</android.support.v7.widget.Toolbar>
```

然后在加载到数据的时候，将这个元素变为可见：

```
share.setVisibility(View.VISIBLE);
```

短信分享示例
---

在实现 UI 之前，我先写了一个简单的分享功能：

```
@OnClick(R.id.share)
void shareAction() {
    BaseShare smsShare = ShareFactory.create("SMS");
    String text = information.getTitle() + ":" + information.getTitle();
    smsShare.share(this, text);
}
```

随后将其重构为简单的工厂模式：

```
public static BaseShare getShareType(String type) {
    switch (type) {
        case "SMS":
            return new SMSShare();
        case "WEIBO":
            return new WeiboShare();
        case "MOMENTS":
            return new MomentsShare();
        case "WECHAT":
            return new WechatShare();
    }
    return null;
}
```


对应于不同的分享类型，都有不同的类来做相应的处理。

使用 Dialog 绘制底部分享
---

在最开始的时候，我使用的是 Dialog 来绘制底部的布局：

```
void showShareDialog() {
    Dialog bottomDialog = new Dialog(this, R.style.BottomDialog);
    View contentView = LayoutInflater.from(this).inflate(R.layout.bottom_share, null);
    bottomDialog.setContentView(contentView);

    ViewGroup.LayoutParams layoutParams = contentView.getLayoutParams();
    layoutParams.width = getResources().getDisplayMetrics().widthPixels;
    contentView.setLayoutParams(layoutParams);

    bottomDialog.getWindow().setGravity(Gravity.BOTTOM);
    bottomDialog.setCanceledOnTouchOutside(true);
    
    bottomDialog.getWindow().setWindowAnimations(R.style.BottomDialog_Animation);
    bottomDialog.show();
    }
```

然后简单地了解了一下动画效果：

```
<style name="BottomDialog">
    <item name="android:windowNoTitle">true</item>
    <item name="android:windowBackground">@android:color/transparent</item>
</style>

<style name="BottomDialog.Animation" parent="Animation.AppCompat.Dialog">
    <item name="android:windowEnterAnimation">@anim/translate_dialog_in</item>
    <item name="android:windowExitAnimation">@anim/translate_dialog_out</item>
</style>
```

对应的动画文件：

translate_dialog_in:

```
<?xml version="1.0" encoding="utf-8"?>
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300"
    android:fromXDelta="0"
    android:fromYDelta="100%"
    android:toXDelta="0"
    android:toYDelta="0">
</translate>
```

translate_dialog_out:

```
<?xml version="1.0" encoding="utf-8"?>
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300"
    android:fromXDelta="0"
    android:fromYDelta="0"
    android:toXDelta="0"
    android:toYDelta="100%">
</translate>
```

但是绘制的时候，出现了一些问题，即 Dialog 在最上面，随后改用 BottomSheetDialog 来绘制。

使用 BottomSheetDialog 绘制分享菜单
---

对应的逻辑变得更加简单了。

```
void showShareDialog() {
    final BottomSheetDialog bottomSheetDialog = new BottomSheetDialog(DetailActivity.this);
    View dialogView = LayoutInflater.from(InformationDetailActivity.this).inflate(R.layout.bottom_share, null);

    dialogView.findViewById(R.id.cancel_share).setOnClickListener(view -> {
        bottomSheetDialog.dismiss();
    });

    bottomSheetDialog.setContentView(dialogView);
    bottomSheetDialog.show();
}
```



