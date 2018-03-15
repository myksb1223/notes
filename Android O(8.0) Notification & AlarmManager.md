## Android O(8.0) Notification & AlarmManager

I haven't known this issue since our user reported bugs about notification.

I tested Android OS version 5.0, 6.0. those worked well. But it didn't work in Oreo.

The reason is Notification Channel is added in Anroid O(Oreo).

So I added some codes about creating channel before sending notification.

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
  NotificationChannel mChannel = notiManager.getNotificationChannel(ClassUpActivity.CHANNEL_ID);
  if(mChannel == null) {
    CharSequence name = context.getString(R.string.Default);
    int importance = NotificationManager.IMPORTANCE_HIGH;
    mChannel = new NotificationChannel(ClassUpActivity.CHANNEL_ID, name, importance);
    notiManager.createNotificationChannel(mChannel);
  }
}
```

I recommend that the code will be added both in MainActivity(starting activity) and just before sending notification.

I was really happy few seconds...
Next issue was it called only once.

When the device became doze mode, service didn't start.

For starting service in doze mode, I added below code.

```java
if (Build.VERSION.SDK_INT > Build.VERSION_CODES.N_MR1) {
  return PendingIntent.getForegroundService(context, (int) model.alarm_id, intent, PendingIntent.FLAG_UPDATE_CURRENT);
}
else {
  return PendingIntent.getService(context, (int) model.alarm_id, intent, PendingIntent.FLAG_UPDATE_CURRENT);
}

```

**`getForegroundService`** is key method!
