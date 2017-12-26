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

