---
layout: post
title:  "Testing Android's AlarmManager"
date:   2015-09-24 10:41:06 +00:00
categories: android
author: Guillermo Orellana
comments: true
tags: android
image: /assets/article_images/2015-09-24-testing-android-alarmmanager/clock.jpg
---
I had to maintain and extend an existing Android application, which I did not really understand deep enough to feel safe
to make big changes that it needed. So I decided to put some scaffolding around in shape of tests, in case everything
decided to fall down. One of the parts I found most difficult to deal with, was Android's AlarmManager related code.

# What is AlarmManager?

AlarmManager is an old friend. It has been available in the Android API literally forever,
starting Android 1.0 (API Level 1)

It is very useful when you want to schedule a task in the future and do not want to involve nasty
Handlers or native Threads which can make your code practicably untestable together with many other
 concerns.

AlarmManager is a system service, thus you need a Context to instantiate it from.

```java
AlarmManager alarmManager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
```

# How to use alarms in Android?

Simple! You create a `PendingIntent` for your favourite service, activity or broadcast and pass it
to the `AlarmManager` instance through the appropriate method. In this example I used `setInexactRepeating`
but you can use whichever you find suitable. Full list as always
[in the docs](http://developer.android.com/reference/android/app/AlarmManager.html#pubmethods).

```java
alarmMgr.setInexactRepeating(
        AlarmManager.ELAPSED_REALTIME_WAKEUP, // type: wake if device asleep
        SystemClock.elapsedRealtime(), // first repetition: asap!
        AlarmManager.INTERVAL_HALF_HOUR, // consequent repetitions: every half hour
        alarmIntent // PendingIntent to launch on alarm
);
```

And that's it! The Android system will call your `PendingIntent` once the timeout has been met.
Bear in mind that for **apps targeting API 19 and higher alarms are no longer exact**, and they can (and
will) be deferred under OS discretion with the purpose of saving battery. This is, if your application
is compiled against KitKat or higher, you cannot rely on the accuracy of any method which is not
`setWindow` or `setExact`.

# How to test alarms in Android?

Such a nice system functionality. You create the alarm, set it up and forget about it. Err but...
How to test it?

Nowadays Robolectric is a de-facto standard in Android unit testing. However, for AlarmManager I
decided to make my tests instrumentation tests, since it is a functionality that relies heavily on
 Android System features. I might rewrite it for Robolectric when 3.1 is out.

## How to check if an alarm is set?

The only access to system alarms in Android is some nasty adb command that dumps the whole system status.
That does not sound right, does it? But don't panic. As [this StackOverflow question]
(http://stackoverflow.com/questions/4556670/how-to-check-if-alarmmanager-already-has-an-alarm-set)
points out, there is a clever way of checking for your pending intents.

```java
private boolean isAlarmSet() {
    Intent intent = MyAlarmReceiver.getTargetIntent(context);
    PendingIntent service = PendingIntent.getService(
            context,
            0,
            intent,
            PendingIntent.FLAG_NO_CREATE
    );
    return service != null;
}
```

Note the flag, `PendingIntent.FLAG_NO_CREATE`. As per [documentation](), this flag will make the
 method call return `null` if the `PendingIntent` is not found in the system. This way we can check
 for our `PendingIntent` in the system, and thus our alarm.

In order for this to work, the `Intent` used in the call must pass the `Intent.filterEquals()` test.
This is, must have same action, data, type, class and categories. Any extra data is ignored. Additionally,
you need to call the same getter, e.g. if you created it calling `getService`, you must use it for
the check as well.

## What could I test?

Well, you will want to test that your alarm is actually set when you want it to be.

```java
@Test
public void testSetAlarm() throws Exception {
    MyAlarmReceiver receiver = new MyAlarmReceiver();
    receiver.setAlarm(context);
    assertThat(isAlarmSet()).isTrue();
}
```

That is not set *automagically*...

```java
@Test
public void testCancelAlarm_notSet() throws Exception {
    MyAlarmReceiver receiver = new MyAlarmReceiver();
    assertThat(isAlarmSet()).isFalse();
}
```

And that you are able to unset it!

```java
@Test
public void testCancelAlarm_set() throws Exception {
    MyAlarmReceiver receiver = new MyAlarmReceiver();

    receiver.setAlarm(context);
    assertThat(isAlarmSet()).isTrue();

    receiver.cancelAlarm(context);
    assertThat(isAlarmSet()).isFalse();
}
```

## Help! All my cancel-related tests are red!

Be careful here. For these tests to work, you have to cancel **both** the alarm and the pending intent.

```java
public void cancelAlarm(Context context) {
	[...]
    if (alarmMgr != null) {
        alarmMgr.cancel(alarmIntent);
    }
    getPendingIntent(context).cancel();
    [...]
}
```

## Clean up before you leave!

Have you ever heard of the [Boy Scouts' rule](http://programmer.97things.oreilly.com/wiki/index.php/The_Boy_Scout_Rule)?
Well, in tests it is quite important to clean up your mess!

```java
@After
public void tearDown() {
    if (receiver != null && receiver.getAlarmManager() != null) {
        receiver.getAlarmManager().cancel(receiver.getAlarmIntent());
    }
}
```

Make sure you cancel any alarm that might have not been cancelled already by the end of the test.
Uncle Bob would be proud!

# Further testing

That should be it for alarm setting and cancelling. But there are many many more things we can test!
For instance, enforcing that we only set up one alarm at the time, that the alarm gets the right
pending intent or what happens when the pending intent is invalidated before alarm goes off.
