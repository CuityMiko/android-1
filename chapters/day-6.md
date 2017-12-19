

自定义 Layout
---

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

```
public class DescriptionLineChart extends RelativeLayout {

	public DescriptionLineChart(Context context, AttributeSet attr) {
		super(context, attr);
		view = LayoutInflater.from(context).inflate(R.layout.chart_layout, this);
	}
}
```

MPAndroidChat Y 轴
---

```
YAxis leftAxis = chart.getAxisLeft();
leftAxis.removeAllLimitLines(); // reset all limit lines to avoid overlapping lines
leftAxis.enableGridDashedLine(10f, 10f, 0f);
leftAxis.setDrawZeroLine(false);
leftAxis.setDrawLimitLinesBehindData(true);
```

X 轴设置
---

```
XAxis xAxis = chart.getXAxis();
xAxis.setPosition(XAxis.XAxisPosition.BOTTOM);
xAxis.setDrawGridLines(true);
xAxis.setGranularity(1f);
xAxis.setLabelCount(5);
IAxisValueFormatter xAxisFormatter = new DateAxisFormatter();
xAxis.setValueFormatter(xAxisFormatter);
xAxis.enableGridDashedLine(10f, 10f, 0f);
```

自定义 LABEL 显示
---

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

