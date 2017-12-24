大前端 Android 开发日记三：使用 ViewHolder 与 XML 编写 UI
===

在之前的两天里，我花了大量的时间在熟悉系统的架构上。因此，在这一天里，我主要花费的时间都是在编写 UI 上，以及对应的一些事件处理。

ViewHolder 的事件
---

按官网的说明：

> 一个 ViewHolder 描述一个项目视图和关于它在 RecyclerView 中的位置的元数据。

对于我使用例子是，我是在我的列表中的条目中使用 ViewHolder，因此每个条目都是单一的 view。这是一个示例的 ViewHolder，我在它的构建类里添加了一个 onClick 事件：

```java
public class RelatedViewHolder extends RecyclerView.ViewHolder {
    public RelatedNewsViewHolder(View view) {
        super(view);
        this.view = view;
        ButterKnife.bind(this, view);

        onClick(view);
    }
}
```

而在这个 Click 事件里，启动了一个新的 Activity：

```
public void onClick(View itemView) {
    itemView.setOnClickListener(view -> {
        Intent intent = new Intent();
        Context context = itemView.getContext();

        intent.setComponent(new ComponentName(context, DetailActivity.class));
        context.startActivity(intent);
    });
}
```

在这里面里，我们会启动一个新的 Activity。

获取 Android Device UUID
---

在实现的过程中，还需要一个获取设备的 UUID。DeviceID 有一个问题时，用户可能不会同意获取，于是还需要一个新的 UUID。为了简化这个逻辑，直接使用 Google 推荐的 UUID，而不是再去用 DeviceID。

```
public class DeviceUtil {
    private static String uniqueID = null;
    private static final String PREF_UNIQUE_ID = "PREF_UNIQUE_ID";

    public synchronized static String getDeviceUUID(Context context) {
        if (uniqueID == null) {
            SharedPreferences sharedPrefs = context.getSharedPreferences(
                    PREF_UNIQUE_ID, Context.MODE_PRIVATE);
            uniqueID = sharedPrefs.getString(PREF_UNIQUE_ID, null);
            if (uniqueID == null) {
                uniqueID = UUID.randomUUID().toString();
                SharedPreferences.Editor editor = sharedPrefs.edit();
                editor.putString(PREF_UNIQUE_ID, uniqueID);
                editor.apply();
            }
        }
        return uniqueID;
    }
}
```

在这个逻辑里，会判断机器上是否存储有之前的 UUID。如果没有的话，就生成一个新的；反之，则从 SharedPreferences 中获取一份。

实现 Android 后退按钮
---

由于打开的是新的页面，因此需要自己做一个 Toolbar，相应的配置代码如下所示：

```
<?xml version="1.0" encoding="utf-8"?>

<android.support.v7.widget.Toolbar xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:toolbar="http://schemas.android.com/apk/res-auto"
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    android:background="?attr/colorPrimaryDark"
    android:gravity="center"
    app:theme="@style/ToolbarColoredBackArrow"
    toolbar:titleTextColor="@color/cmb_white">

    <TextView
        android:id="@+id/toolbar_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="xxx" />

</android.support.v7.widget.Toolbar>
```

接着，定制一个新的返回按钮标题：

```
<style name="ToolbarColoredBackArrow" parent="AppTheme">
    <item name="android:textColorSecondary">#FFFFFF</item>
</style>
```  

然后，导入这个 xml：  

```
<include layout="@layout/toolbar" />
```

同时在我们的 Activity 中添加对应的事件处理：

```
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    if (item.getItemId() == android.R.id.home) {
        onBackPressed();
        return true;
    }
    return super.onOptionsItemSelected(item);
}
```

矢量图形：Vector Path
---

还遇到一个很有遇到的问题是，可以在 Android 中使用类似于 SVG 的 Vector：

```
<vector xmlns:android="http://schemas.android.com/apk/res/android"
     android:height="64dp"
     android:width="64dp"
     android:viewportHeight="600"
     android:viewportWidth="600" >
     <group
         android:name="rotationGroup"
         android:pivotX="300.0"
         android:pivotY="300.0"
         android:rotation="45.0" >
         <path
             android:name="v"
             android:fillColor="#000000"
             android:pathData="M300,70 l 0,-70 70,70 0,0 -70,70z" />
     </group>
 </vector>
```

它也是类似的使用 Path，来描述 UI。

ScrollView
---

最后，花费了一些时间在编写一个 ScrollView 上，结合了之前上面的 Toolbar 等内容：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/cmb_white"
    android:orientation="vertical">

    <include layout="@layout/toolbar" />

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scrollbars="none">

        <LinearLayout
            android:id="@+id/detail"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">

        </LinearLayout>
    </ScrollView>
</LinearLayout>
```

