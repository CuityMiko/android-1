1. unused filted
2. 测试 null
3. info -> information
4. uuid
5. toolbar Icon 
6. UI 组件

重构 DeviceUUID 存储
---

使用公共类：

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

Android 后退按钮图片
---

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

```
@PerActivity
@Component(dependencies = ApplicationComponent.class, modules = HomeModule.class)
public interface HomeComponent {

    void inject(HomeFragment homeFragment);

    void inject(DetailActivity detailActivity);
}
```

intent 传递列表
---

```
public class Model implements Serializable {

}
```

```
intent.putExtra("model", (Serializable) model);
```

```
List<Model> analyses = null;
Bundle extras = getIntent().getExtras();
if (extras == null) {
    onBackPressed();
} else {
    analyses = (List<Model>) getIntent().getSerializableExtra("model");
}
```

自定义 RelativeLayout 组件
---



右箭头
---

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

