
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


大前端 Android 开发日记六：使用 MPAndroidChat 开发图表应用
===

在完成了基本的业务功能之后，我开始去画相应的图表。这不是一件简单的事，尽管已经有了 MPAndroidChart 这样的图表工具。但是显然，它带来的问题，可能比解决的问题还多。

这一天做的事情比较少，主要是在做（学习，边做边学）图表相关的内容。

使用 Java 编写 Layout
---

在这之前，我完全不知道，怎么用 Java 代码去写一个自定义的布局，即如下的示例：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/cmb_white"
   
    <com.phodal.app.LineChart
        android:id="@+id/line_chart"
        android:layout_width="match_parent"
        android:layout_height="160dp" />

</LinearLayout>
```

这里的 LineChart 就是一个自定义的布局，下面就是对应的类。

```
public class LineChart extends RelativeLayout {

	public LineChart(Context context, AttributeSet attr) {
		super(context, attr);
		view = LayoutInflater.from(context).inflate(R.layout.chart_layout, this);
	}
}
```

在这里，我们就可以进行相应的元素操作了。

MPAndroidChat Y 轴
---

然后就是一些 MPAndroidChat 相应的设置。在 MPAndroidChat 中，传统的 Y 轴是叫 LeftAxis：

```
YAxis leftAxis = chart.getAxisLeft();
leftAxis.removeAllLimitLines(); 
leftAxis.enableGridDashedLine(10f, 10f, 0f);
leftAxis.setDrawZeroLine(false);
leftAxis.setDrawLimitLinesBehindData(true);
```

MPAndroidChat X 轴设置
---

以及 X 轴相关的一些设备：

```
XAxis xAxis = chart.getXAxis();
xAxis.setPosition(XAxis.XAxisPosition.BOTTOM);
xAxis.setDrawGridLines(true);
xAxis.setGranularity(1f);
IAxisValueFormatter CustomAxisFormatter = new DateAxisFormatter();
xAxis.setValueFormatter(xAxisFormatter);
xAxis.enableGridDashedLine(10f, 10f, 0f);
```

自定义 LABEL 显示
---

对应的，还有相应的格式化数据的逻辑：

```
public class CustomAxisFormatter implements IAxisValueFormatter {
    @Override
    public String getFormattedValue(float value, AxisBase axis) {
        Date date = new Date((long) value);
        SimpleDateFormat sdf = new SimpleDateFormat("MM-dd", Locale.CHINA);
        return sdf.format(date);
    }
}
```


大前端 Android 开发日记七：MPAndroidChat 填坑笔记
===

继续上一天的 MPAndroidChat 填坑记录。

MPAndroidChat 自定义 Marker
---

首先，是自定义用来 Highlight 的 Marker。代码如下所示：

```
@SuppressLint("ViewConstructor")
public class DescriptionChartMarkerView extends MarkerView {
    private TextView content;
    private int startIndex;

    public DescriptionChartMarkerView(Context context, int layoutResource) {
        super(context, layoutResource);
        content = (TextView) findViewById(R.id.view);
    }

    @SuppressLint("DefaultLocale")
    @Override
    public void refreshContent(Entry e, Highlight highlight) {
        super.refreshContent(e, highlight);
        content.setText(String.format("%d", startIndex + 1));
        startIndex++;
    }

    public void setStartIndex(int startIndex) {
        this.startIndex = startIndex;
    }
}
```

Android Vector 圆圈
---

在实现的过程中，需要画一个圆圈，也就有了：

```
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">

    <solid
        android:color="@color/green"/>

    <size
        android:width="@dimen/text_size_14"
        android:height="@dimen/text_size_14"/>
</shape>
```


MPAndroidChat 生成位置
---

一般来说，我们可以通过下面的代码来生成  Position 的位置，但是好似是不工作的：

```
chart.getTransformer(chart.getLineData().mDataSets.get(0).getAxisDependency()).getPixelForValues(highlights[index].getX(), highlights[index].getY());
```

添加子布局
---

在上一步获取位置之后，接下来就是将对应的点绘制在坐标上：

```
View marker = LayoutInflater.from(getContext()).inflate(R.layout.marker, view, false);
RelativeLayout.LayoutParams params  = (RelativeLayout.LayoutParams) marker.getLayoutParams();

TextView textView = (TextView) marker.findViewById(R.id.text_view);
textView.setText(String.format("%d", i));

params.leftMargin = 20
params.topMargin = 20

marker.setLayoutParams(params);
view.addView(marker);
```

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

大前端 Android 开发日记 11：日历创建
===

在写了一些业务代码之后，终于有机会去处理一些坑了——日历。业务上的需求是，用户可以添加对应的日历提醒，因此就开始这一天的挖坑之旅了。

日历权限
---

首先，在 AndroidManifest.xml 中加入 ``READ_CALENDAR`` 权限。为了删除、插入或者更新日历数据，则需要添加 ``WRITE_CALENDAR`` 权限

```
<uses-permission android:name="android.permission.READ_CALENDAR" />
<uses-permission android:name="android.permission.WRITE_CALENDAR" />
```

权限判断
---

然后添加一些权限判断代码：

```
private static boolean hasReadCalendarPermissions(Activity activity) {
    return PackageManager.PERMISSION_GRANTED == ContextCompat.checkSelfPermission(activity, Manifest.permission.READ_CALENDAR);
}
```


请求权限
---

对于高版本的 Android 系统来说，还需要动态地请求权限：

```
public static void requestCalendarPermissions(Activity activity) {
    if (Build.VERSION.SDK_INT >= 23 && !CalenderHelper.hasReadCalendarPermissions(activity)) {
        int request = ContextCompat.checkSelfPermission(activity, Manifest.permission.WRITE_CALENDAR);
        if (request != PackageManager.PERMISSION_GRANTED) {
            PERMISSION_REQUEST_CODE++;
            ActivityCompat.requestPermissions(activity, new String[]{
                    Manifest.permission.WRITE_CALENDAR,
                    Manifest.permission.READ_CALENDAR
            }, PERMISSION_REQUEST_CODE);
        }

    }
}
```

请求权限回调
---

请求完权限后，可以通过 ``onRequestPermissionsResult`` 来知道，是否能添加日历。

```
@SuppressLint("ShowToast")
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);

    if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_DENIED) {
        Toast.makeText(this.getContext(), "请求日历权限失败", Toast.LENGTH_LONG);
    } else if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
        Toast.makeText(this.getContext(), "请求日历权限成功，请重新添加!", Toast.LENGTH_LONG);
    }
}
```

添加前检测权限
---

如果用户如果权限，并且想添加日历事件的时候，我们仍然需要做一个检测：

```
public static void addEvent(Context context, CalendarModel calendar) {
    if (!CalenderHelper.hasReadCalendarPermissions((Activity) context)) {
        CalenderHelper.requestCalendarPermissions((Activity) context);
    }

    if (CalenderHelper.hasReadCalendarPermissions((Activity) context)) {
        addEventByCalendar(context, calendar);
    }
}
```

创建日历事件
---

在结束了权限相关的话题之后，我们就可以创建一个日历事件了。按照 Google Android 官方的文档，就可以拼出我们的事件：

```
ContentValues event = new ContentValues();
Calendar mCalendar = Calendar.getInstance();
mCalendar.setTime(beginTime);
long start = mCalendar.getTime().getTime();
long end = mCalendar.getTime().getTime();

event.put(CalendarContract.Events.CALENDAR_ID, 1);
event.put(CalendarContract.Events.TITLE, calendar.getTitle());
event.put(CalendarContract.Events.DESCRIPTION, "Description");
event.put(CalendarContract.Events.DTSTART, start);
event.put(CalendarContract.Events.DTEND, end);
event.put(CalendarContract.Events.HAS_ALARM, 1);
event.put(CalendarContract.Events.EVENT_TIMEZONE, "Asia/Shanghai");

Uri newEvent = context.getContentResolver().insert(Uri.parse(CALENDER_EVENT_URL), event);
if (newEvent == null) {
    return;
}
```

添加提醒
---

添加完日历之后，我们可以顺便添加一个提醒：

```
ContentValues values = new ContentValues();
values.put(CalendarContract.Reminders.EVENT_ID, ContentUris.parseId(newEvent));

values.put(CalendarContract.Reminders.MINUTES, 5);
values.put(CalendarContract.Reminders.METHOD, CalendarContract.Reminders.METHOD_ALERT);
Uri uri = context.getContentResolver().insert(Uri.parse(CALENDER_REMINDER_URL), values);
if (uri == null) {
    return;
}
```

获取所有的日历
---

添加完日历之后，就可以获取日历来测试了。按照官方的示例，写了以下的代码：

```
ContentUris.appendId(uriBuilder, eStartDate.getTimeInMillis());
ContentUris.appendId(uriBuilder, eEndDate.getTimeInMillis());

Uri uri = uriBuilder.build();

String selection = "((" + CalendarContract.Instances.BEGIN + " >= " + eStartDate.getTimeInMillis() + ") " +
        "AND (" + CalendarContract.Instances.END + " <= " + eEndDate.getTimeInMillis() + ") " +
        "AND (" + CalendarContract.Instances.VISIBLE + " = 1) )";

cursor = cr.query(uri, new String[]{
        CalendarContract.Instances.EVENT_ID,
        CalendarContract.Instances.TITLE,
        CalendarContract.Instances.DESCRIPTION,
        CalendarContract.Instances.BEGIN,
        CalendarContract.Instances.END
}, selection, null, null);
ArrayList<String> nameOfEvent = new ArrayList<>();

if (cursor != null && cursor.moveToFirst()) {
    String CNames[] = new String[cursor.getCount()];
    for (int i = 0; i < CNames.length; i++) {
        nameOfEvent.add(cursor.getString(1));
        CNames[i] = cursor.getString(1);
        cursor.moveToNext();
    }
    cursor.close();
}

return nameOfEvent;
```

手动拼写 QUERY，想死的感觉都有了。尝试了很多之后，终于写出可以工作的代码了。


大前端 Android 开发日记 12：删除日历、状态变化
===

有了上一天的基础，就可以一步步地实现业务代码了。

合并日历的状态
---

由于不熟悉 rxJava，不知道怎么用 Java 来合并类似于 JavaScript 中的多个 Promise 的东西。找了项目上的 Android 大牛，写了如下的代码：

```
Flowable.fromCallable(() -> CalenderHelper.findEventCalendars(context, date))
        .zipWith(getCalendar(date), (localEvents, remoteCalendars) -> {
            List<CalendarModel> calendarList = remoteCalendars.getData();

            ...

            return calendarList;
        })
        .compose(RxjavaUtils.flowableIoTransformer())
        .subscribe(callback::onLoadCalendarComplete, callback::onLoadCalendarFailed);
```

首先从本地获取指定日志的所有日历，然后再通过 ``getCalendar`` 方法从远程获取日历，随后在 ``zipWith`` 方法中，来 merge 两个日历。

删除日历
---

删除日历倒是比之前的内容简单一些，我们只需要找到对应的日历事件的 ID，然后就可以删除了。

```
public static void removeEvent(Context context, int indexId, int eventID) {
    Uri deleteUri;
    deleteUri = ContentUris.withAppendedId(Uri.parse(calenderEventUrl), eventID);
    int rows = context.getContentResolver().delete(deleteUri, null, null);
    Timber.d("Rows deleted: " + rows);

    RemoveCalenderEvent removeCalenderEvent = new RemoveCalenderEvent(indexId, eventID);
    EventBus.getDefault().post(removeCalenderEvent);
}
```

在这个过程中，最初我犯了一个错误，那就是最开始我找的是日历的 CALENDAR_ID，而不是 EVENT_ID。


实时更新日历状态
---

创建日历的时候，发出一个对应的事件：

```
CreateCalenderEvent createCalenderEvent = new CreateCalenderEvent(calendar.getId(), (int) eventId);
EventBus.getDefault().post(createCalenderEvent);
```

随后在 Activity 里，来接收相应的事件：

```
@SuppressLint("ShowToast")
@Subscribe
public void onRemoveCalendarThread(RemoveCalenderEvent removeCalenderEvent) {
    Toast.makeText(this.getContext(), "取消提醒成功 ", Toast.LENGTH_LONG);
    int id = removeCalenderEvent.getId();
    int eventID = removeCalenderEvent.getEventId();
    Timber.d("取消提醒成功 " + id);

    calendarAdapter.updateItem(id, eventID, "remove");
}
```

接着在我们的 Adapter 里更新对应的日历，然后使用 ``notifyItemChanged`` 来刷新整个 item 的视图。

```
public void updateItem(int updateItemId, int eventID, String updateType) {
    CalendarModel item = null;

    ...

    if (updateType.equals("create")) {
        item.setEventId(eventID);
    } else if (updateType.equals("remove")) {
        item.setEventId(0);
    }

    calendarList.set(index, item);
    notifyItemChanged(index);
}
```


大前端 Android 开发日记 13：动态更新日历状态
===

继续之前的日志开发，在之前的文章中，我们已经可以从系统日历里读取、删除、添加日历了。

在这一篇里，我要总结提：每当我添加或者删除日历的时候，我需要同时更新页面上的元素。因此，在这里我们分为了四个步骤：

 1. 使用 EventBus 创建通知事件
 2. 接收事件，并传递给 Adapter
 3. 更新 Item
 4. 更新状态

动态更新日历状态
---

### 1.使用 EventBus 创建通知事件


由于之前已经介绍过 EventBus 的相关内容，这里简单地列一下代码就好了：

```
CreateCalenderSuccessEvent createCalenderSuccessEvent = new CreateCalenderSuccessEvent(calendar.getId(), (int) eventId);
        EventBus.getDefault().post(createCalenderSuccessEvent);
```

### 2.接收事件，并传递给 Adapter

随后在我们的 Activity 中接收相应的事件，然后调用 Adapter 中的方法：

```
@SuppressLint("ShowToast")
@Subscribe
public void onCreateCalendarThread(CreateCalenderSuccessEvent createCalenderSuccessEvent) {
    ...

    calendarAdapter.updateItem(id,  eventID, "create");
}
```

### 3. 更新 Item

然后根据传过来的更新类型，我们来为对应的 item 修改 eventid 的值：

```
public void updateItem(int updateItemId, int eventID, String updateType) {
    CalendarModel item = null;
    ...

    if (updateType.equals("create")) {
        item.setEventId(eventID);
    } else if (updateType.equals("remove")) {
        item.setEventId(0);
    }

    calendarList.set(index, item);
    notifyItemChanged(index);
}
```

### 4. 更新状态

最近根据有没有 eventid，来决定是否显示的内容：

```
if (calendar.getEventId() != 0) {
    calendar_reminder_.setText(R.string.cancel);
} else {
    text.setText(R.string.title);
}
```

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


大前端 Android 开发日记 14：Android WebView 默认 margin 样式问题
===

这一也没啥做的，不过也算是修了一个 CSS 的样式问题。

这个问题的主要来源是，后端返回的 HTML 是不带样式的，而默认的 WebView 会为 Div 标签加上 8px 的 padding，也就是说没有 CSS Reset。这个时候，就需要手动加一个 Reset 效果了，于是就写了一个简单的方法来做这件事。顺便带页面的内容，包在了 body 中。

```
public class WebViewUtil {
    public static String removeDefaultStyle(String originContent) {
        return "<body style='margin:0;padding:0;background-color: transparent;'>" + originContent + "</body>";
    }
}
```

Reset 的 CSS 的内容就是：

```
margin:0;
padding:0;
```

然后在一次新的测试中，发现了样式中又默认了白色背景，于是就加了一个背景透明 ``background-color: transparent;``。

