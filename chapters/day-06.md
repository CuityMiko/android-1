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

