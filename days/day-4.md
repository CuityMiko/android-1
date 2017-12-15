
Android Day 4：Toolbar 问题及测试
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


