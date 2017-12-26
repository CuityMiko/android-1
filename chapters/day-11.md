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

