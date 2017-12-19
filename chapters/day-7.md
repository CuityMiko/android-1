
===

MPAndroidChat 自定义 Marker
---

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

圆圈
---

```
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">

    <solid
        android:color="@color/cmb_green"/>

    <size
        android:width="@dimen/text_size_14"
        android:height="@dimen/text_size_14"/>
</shape>
```


Charting 设置位置
---

```
chart.getTransformer(chart.getLineData().mDataSets.get(0).getAxisDependency()).getPixelForValues(highlights[index].getX(), highlights[index].getY());
```

添加子布局
---

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