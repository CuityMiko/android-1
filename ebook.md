
大前端 Android 开发日记一：了解基本业务与技术栈
===

初来的第一天，由项目的开发人员及业务人员，介绍了项目的业务知识。由于产权限制，这里就不多说业务相关的知识了。

简单的记录一下，项目所用到的 Android 技术栈。

**Picasso**
---

Picasso 是Square公司开源的一个Android图形缓存库，可以实现图片下载和缓存功能。官网地址: http://square.github.io/picasso/

其使用方式也很简单：

```
Picasso.with(context).load("http://i.imgur.com/DvpvklR.png").into(imageView);
```

**Retrofit2**
---

Retrofit2 是一个用于 Android 和 Java 平台的类型安全的网络框架。

可以支持这种路径的参数获取：

```
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```

以及：

```
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```

我们唯一要做的比较麻烦的可能是构建模型。

**otto**
---

Otto 是 square 公司出的一个事件库（pub/sub模式），用来简化应用程序组件之间的通讯。如下是一个简单的例子：

```
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View rootView = inflater.inflate(R.layout.fragment_main, container, false);
    View button = rootView.findViewById(R.id.fragmentbutton);
    button.setOnClickListener(new View.OnClickListener() {

        @Override
        public void onClick(View v) {
            bus.post("Hello from the Fragment");
        }
    });
    bus.register(this);
    return rootView;
}

@Subscribe
public void getMessage(String message) {
    Toast.makeText(getActivity(), message, Toast.LENGTH_SHORT).show();
}
```

**RxJava2**
---

RxJava 是一个 Java VM implementation of Reactive Extensions: a library for composing asynchronous and event-based programs by using observable sequences。

> 一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库。

总之就是这样的：

```
Observable.zip(getCricketFansObservable(), getFootballFansObservable(),
        new BiFunction<List<User>, List<User>, List<User>>() {
            @Override
            public List<User> apply(List<User> cricketFans, List<User> footballFans) throws Exception {
                return Utils.filterUserWhoLovesBoth(cricketFans, footballFans);
            }
        })
        // Run on a background thread
        .subscribeOn(Schedulers.io())
        // Be notified on the main thread
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(getObserver());
```

ButterKnife
---

ButterKnife 是一个使用注解方式来为 Android 中的 View 视图绑定字段和方法，能通过自动解析注解来搜索资源文件并赋值给 Activity 中的字段。

简单的来说就是简化我们的绑定：

```
class ExampleActivity extends Activity {
  @BindView(R.id.title) TextView title;
  @BindView(R.id.subtitle) TextView subtitle;
  @BindView(R.id.footer) TextView footer;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.bind(this);
    // TODO Use fields...
  }
}
```

Dagger
---

Dagger 是目前最流行的专为 Android 设计的依赖注入函数库。

就是有些复杂：

```
@Module(injects = {App.class})
public class AppModule {

    private App app;

    public AppModule(App app) {
        this.app = app;
    }

    @Provides @Singleton public Context provideApplicationContext() {
        return app;
    }
}
```



大前端 Android 开发日记二：编写 MVP + 依赖流入的 Activity
===

在这一天里，我接到了一张新的业务卡，从技术上来说就是：从列表页打开一个详情页。

而详情页的入口有两部分，一个是在首页，一个是在列表页。而首页是一个独立的 APK，因此在启动方式就是采用 Actvitiy 来启动。

因此，我做了一个简单的 Task，来做这个卡：

1. 创建一个空白的详情页
2. 从首页打开详情页
3. 将参数传递到详情页
4. 在详情页去获取数据
5. 展示数据
6. 优化 UI（留给第二天）

1.创建一个空白页
---

这一步就比较简单了，网上找个 Hello, world 放置一下就可以了：

首先在 ``AndroidManifest.xml`` 中添加我们的入口 actvity：

```
       <activity
          android:name=".DetailActivity"
          android:label="@string/title_activity_main" >

          <intent-filter>
             <action android:name="android.intent.action.MAIN" />
             <category android:name="android.intent.category.LAUNCHER"/>
          </intent-filter>

       </activity>
```

然后创建好这个 activity：

```
package com.example.helloworld;

import android.os.Bundle;
import android.app.Activity;
import android.view.Menu;
import android.view.MenuItem;
import android.support.v4.app.NavUtils;

public class MainActivity extends Activity {

   @Override
   public void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_main);
   }

   @Override
   public boolean onCreateOptionsMenu(Menu menu) {
      getMenuInflater().inflate(R.menu.activity_main, menu);
      return true;
   }
}
```

以及对应的 Layout：

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:tools="http://schemas.android.com/tools"
   android:layout_width="match_parent"
   android:layout_height="match_parent" >

   <TextView
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_centerHorizontal="true"
      android:layout_centerVertical="true"
      android:padding="@dimen/padding_medium"
      android:text="@string/hello_world"
      tools:context=".MainActivity" />

</RelativeLayout>
```


2.首页打开列表页
---

这部分的代码比较简单，如下所示：

```
Intent intent = new Intent();
intent.setComponent(new ComponentName("com.example", "com.example.MyExampleActivity"));
startActivity(intent);
```

不过，我一开始写的代码是错的。

3.将参数传递到详情页
---

这一步也不难，就是先 putExtras 再 getExtras：

```
Intent intent = new Intent();
intent.setComponent(new ComponentName("com.example", "com.example.MyExampleActivity"));
intent.putExtra("id", id);            
startActivity(intent);
```

再在我们的 Activity 中获取参数：

```
    String newString;
    Bundle extras = getIntent().getExtras();
    if (extras == null) {
        newString = "123";
    } else {
        newString = extras.getString("id");
    }
```


4.在详情页去获取数据
---

我本来以为这一部分很简单，但是没想到的是我们的 Android 项目是遵循 Google 推荐的 MVP 架构。可以在 Google Samples 看到一个相关的示例：[todo-mvp](https://github.com/googlesamples/android-architecture/tree/todo-mvp)

按照这个架构，我们需要有：

 - Model，在其中定义我们的数据结构，对应于后台的 API
 - View，定义显示相关的逻辑，具体逻辑会由 Activity 实现
 - Presenter，处理事件，检索 Model 获取数据等等
 - Repository，作为数据源，来处理远程或者本地的数据

让我们看看这个 Todo 的 Model：

```
@Entity(tableName = "tasks")
public final class Task {
    @PrimaryKey
    @NonNull
    @ColumnInfo(name = "entryid")
    private final String mId;

    @Nullable
    @ColumnInfo(name = "title")
    private final String mTitle;

    @Nullable
    @ColumnInfo(name = "description")
    private final String mDescription;

    @ColumnInfo(name = "completed")
    private final boolean mCompleted;

    @Ignore
    public Task(@Nullable String title, @Nullable String description) {
        this(title, description, UUID.randomUUID().toString(), false);
    }
	...
}
```	

如下是一个 View 的示例：

```
package com.example.android.architecture.blueprints.todoapp;

public interface BaseView<T> {

    void setPresenter(T presenter);

}
```

下面是一个 Presenter 的示例：

```
package com.example.android.architecture.blueprints.todoapp;

public interface BasePresenter {

    void start();

}
```

对应的还有处理请求的 Repository：

```
public class TasksRepository implements TasksDataSource {

	...
	    @Override
    public void getTasks(@NonNull final LoadTasksCallback callback) {

	})
}
```

而为了使用 Dagger 来做依赖注入，还需要额外的：

 - Module
 - Component

如下是一个 Dagger 的简单示例（来源：[使用Dagger 2进行依赖注入](http://codethink.me/2015/08/06/dependency-injection-with-dagger-2/)）：

Module 用于提供依赖：

```
@Module
public class ActivityModule {
    @Provides UserModel provideUserModel() {
        return new UserModel();
    }
}
```

component 负责连接提供依赖和消费依赖对象：

```
@Component(modules = ActivityModule.class)
public interface ActivityComponent {
    void inject(MainActivity activity);
}
```

然后就可以愉快地使用了：

```
    @Inject
    UserModel userModel;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
    	...
        mActivityComponent = DaggerActivityComponent.builder().activityModule(new ActivityModule()).build();
        mActivityComponent.inject(this);
    })
```        

5.展示数据
---

当我们 Activity 将会继承自上面的 View，以及实现对应的逻辑：

```
@Override
public void showDetail(TaskDetailModel task) {
    ViewHolder viewHolder = new ViewHolder(content);
    ViewHolder.populate(task);
}
```    

我们将对应的展示交由 ViewHoler 来实现：

```
public class ViewHolder extends RecyclerView.ViewHolder {

    private final View view;


    @BindView(R.id.webview)
    WebView webview;

    public InformationDetailViewHolder(View view) {
        super(view);
        this.view = view;
        ButterKnife.bind(this, this.view);
    }

    void populate(InformationDetailModel InformationDetail) {
        webview.loadData(InformationDetail.getContent(), "text/html; charset=utf-8", "UTF-8");
    }
}
```

就是这么复杂。

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


大前端 Android 开发日记四：Toolbar 问题及测试
===

在之前的两天里，已经实现了大部分的功能。但是，仍然遇到了一些 Toolbar 的问题，除了努力地解决这个问题之外，还写了几个简单的测试。

模块化 APK 的 Toolbar 后退按钮
---

由于应用程序采用的是类似于 RePlugin 的插件化机制。因为在使用样式来将 Toolbar 改为白色的时候，在另外一个 APK 里没有对应的资源。于是，便想着将 Toolbar 改成了个组件：

```
<RelativeLayout
    android:layout_width="match_parent"
    android:layout_height="56dp"
    android:background="@color/black"
    android:gravity="center_vertical">

    <include layout="@layout/back_button" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:padding="@dimen/length_16"
        android:text="@string/something"/>

</RelativeLayout>
```

对应的后退按钮：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fast_back"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:gravity="center"
    android:orientation="horizontal">

    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingEnd="@dimen/length_24"
        android:paddingStart="@dimen/length_16"
        android:paddingTop="@dimen/length_16"
        android:paddingBottom="@dimen/length_16"
        android:src="@drawable/back_icon"/>

</LinearLayout>
```

这样在另外一个 APK 中，只需要有相应的图片资源即可。在 Code Diff 的时候，被告知这个可以用图片来实现——做的时候，忘记了这个。只能明天再去改吧。

Android 处理 Toolbar 后退
---

对应的处理后退的逻辑，也由：

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

变成了简单的后退了。

```
@OnClick(R.id.fast_back)
void clickBackText() {
    onBackPressed();
}
```

测试
---

完成了对 Toolbar 的控制之后，我便写了一个测试：

```
    @Mock
    private DetailView detailView;
    @Mock
    private DetailRepository detailRepository;

    private DetailPresenter detailPresenter;

	@Test
	public void shouldShowDetail() throws Exception {
	    DetailModel detail = mock(DetailModel.class);
	    when(detail.getText()).thenReturn(anyListOf(Related.class));

	    detailPresenter.onLoadDetailSuccess(detail);
	    verify(detailView).showDetail(any(DetailModel.class));
	}
```

在这个测试里，简单的测试了一个回调成功时，会调用显示详情的逻辑。



大前端 Android 开发日记五：代码重构
===

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

