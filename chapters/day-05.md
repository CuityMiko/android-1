大前端 Android 开发日记五：代码重构
===

本来在上一天里，我已经完成了业务的功能。在 Code Review 的时候，代码中有一些不符合 Android 的实践，又或者是有更好的方式。

这些不好的 Code Smell，比如说：

1. 在 AndroidManifest.xml 中有没有使用的 Intent Filted
2. 测试的时候，漏掉了 null 的情况
3. 其它代码中使用了单词缩写，应该改为全称
4. 代码中已经有一个 SharedPreferences 工具类
5. Toolbar （回退按钮）的图标可以在 Java 代码里设置

花了一个小时重构完之后，我便开始写新的业务卡。

Android Manifest.xml
---

由于事先不是很了解 Android 的一些 xml 配置，便直接复制了原先的 MainActivity。当然，这些代码是多余的：

```
<intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```

按 Android 官方的文档：

> 应用的核心组件（例如其 Activity、服务和广播接收器）由 intent 激活。Intent 是一系列用于描述所需操作的信息（Intent 对象），其中包括要执行操作的数据、应执行操作的组件类别以及其他相关说明。Android 系统会查找合适的组件来响应 intent，根据需要启动组件的新实例，并将其传递到 Intent 对象。

> 组件将通过 intent 过滤器公布它们可响应的 intent 类型。由于Android 系统在启动某组件之前必须了解该组件可以处理的 intent，因此 intent 过滤器在清单中被指定为 <intent-filter> 元素。一个组件可有任意数量的过滤器，其中每个过滤器描述一种不同的功能。

相关的解释如下

 - android.intent.action.MAIN 决定应用程序最先启动的Activity
 - android.intent.category.LAUNCHER 决定应用程序是否显示在程序列表里

因此，便删了这两行代码。

重构 DeviceUUID 存储
---

SharedPreferences API 是一种类似于 LocalStorage 的读写键值接口。最大的区别是，SharedPreferences 是以文件名来保存的。

这就意味着我们需要从某个地方来放置一个文件名相关的变量，因此更好的方式是有一个 SharedPreferences 的公共类。它负责做这些相关的事件：

```
@SuppressLint("ApplySharedPref")
public void setDeviceUUIDPref(String value) {
    pref.edit().putString(PREF_UNIQUE_ID, value).commit();
}

@SuppressLint("ApplySharedPref")
public String getDeviceUUIDPref() {
    return pref.getString(PREF_UNIQUE_ID, null);
}
```

然后，我们从来读取数据：

```
public class DeviceUtil {
    private static String uniqueID = null;

    public synchronized static String getDeviceUUID(Context context) {
        PreferencesHelper sharedPrefs = new PreferencesHelper(context);
        uniqueID = sharedPrefs.getDeviceUUIDPref();
        if (uniqueID == null) {
            uniqueID = UUID.randomUUID().toString();
            sharedPrefs.setDeviceUUIDPref(uniqueID);
        }
        return uniqueID;
    }
}
```

这样就可以保证，我们读取的是同一个文件。

Android 后退按钮图片
---

再回到那个 Android 后退按钮的图片问题，可以直接使用：

```
actionBar.setHomeAsUpIndicator(R.drawable.back_icon);
```

这样就可以设置上图片了：

```
setSupportActionBar(toolbar);
ActionBar actionBar = getSupportActionBar();
if (actionBar != null) {
    actionBar.setDisplayHomeAsUpEnabled(true);
    actionBar.setDisplayShowTitleEnabled(false);
    actionBar.setDisplayShowHomeEnabled(true);
    actionBar.setHomeAsUpIndicator(R.drawable.back_icon);
}
```

Dagger 的 Component 注入多个
---

因为新建了一个 Activity，就需要注入新的 Component：

```
@PerActivity
@Component(dependencies = ApplicationComponent.class, modules = HomeModule.class)
public interface HomeComponent {

    void inject(HomeFragment homeFragment);

    void inject(DetailActivity detailActivity);
}
```

Android intent 传递 LIST
---

在实现新业务的过程中，还遇到一个使用 intent 传递 LIST 的情形，一种比较简单的方式是使用 Serializable 来实现：

```
public class Model implements Serializable {

}
```

然后 put 进去：

```
intent.putExtra("model", (Serializable) model);
```

接着，再用的地方转换一下，读取出来：

```
List<Model> model = null;
Bundle extras = getIntent().getExtras();
if (extras == null) {
    onBackPressed();
} else {
    model = (List<Model>) getIntent().getSerializableExtra("model");
}
```


Android 列表箭头图片
---

在实现一个列表的时候，用到了一个箭头。这个箭头正好要水平居中，你懂的。就有了如下的 XML：

```
<android.support.v7.widget.AppCompatImageView
    android:id="@+id/chevron_right"
    android:visibility="gone"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_alignParentEnd="true"
    android:layout_centerVertical="true"
    app:srcCompat="@drawable/ic_chevron_right" />
```

