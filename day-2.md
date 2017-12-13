Android 日记二：编写 MVP + 依赖流入的 Activity
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

1. 创建一个空白页
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


2. 首页打开列表页
---

这部分的代码比较简单，如下所示：

```
Intent intent = new Intent();
intent.setComponent(new ComponentName("com.example", "com.example.MyExampleActivity"));
startActivity(intent);
```

不过，我一开始写的代码是错的。

3. 将参数传递到详情页
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


4. 在详情页去获取数据
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

5. 展示数据
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
