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


