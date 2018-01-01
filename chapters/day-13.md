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
