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
