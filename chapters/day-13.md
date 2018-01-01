大前端 Android 开发日记 13：动态更新日历状态
===


### 1.使用 EventBus 创建通知事件

```
CreateCalenderSuccessEvent createCalenderSuccessEvent = new CreateCalenderSuccessEvent(calendar.getId(), (int) eventId);
        EventBus.getDefault().post(createCalenderSuccessEvent);
```

### 2.接收事件，并传递给 Adapter

```
@SuppressLint("ShowToast")
@Subscribe
public void onCreateCalendarThread(CreateCalenderSuccessEvent createCalenderSuccessEvent) {
    Toast.makeText(this.getContext(), R.string.already_add_event, Toast.LENGTH_SHORT).show();
    int id = createCalenderSuccessEvent.getId();
    int eventID = createCalenderSuccessEvent.getEventId();
    Timber.d("日历创建成功 " + id);

    calendarAdapter.updateItem(id,  eventID, "create");
}
```

### 3. 更新 Item

```
public void updateItem(int updateItemId, int eventID, String updateType) {
    CalendarModel item = null;
    int index = -1;
    for (int i = 0, size = calendarList.size(); i < size; i++) {
        if (calendarList.get(i).getId() == updateItemId) {
            index = i;
            item = calendarList.get(i);
            break;
        }
    }
    if (index < 0) {
        return;
    }

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

```
if (calendar.getEventId() != 0) {
    calendar_reminder_.setText(R.string.cancel);
} else {
    text.setText(R.string.title);
}
```
