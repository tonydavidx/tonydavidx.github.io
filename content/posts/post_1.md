---
title: "How to Run Background Tasks in Flutter"
date: 2020-09-15T11:30:03+00:00
# weight: 1
aliases: ["/first"]
tags: ["flutter"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "learn how to use android alarm manager package to run your dart code in background"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
cover:
    image: "static/alarmmanager.jpg" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page

---

flutter apps wont run code when the screen is not active or  phone is locked if you switch to different app your flutter app will stop running.  so incase you want to trigger a function at certain time or lets say you have a weather app and you want to update weather every one hour automatically  you can do that using android alarm manager package.

First I created flutter demo project. inside it i have a switch button at the center to start and stop the alarm timer. you can download the starter files from this GitHub [link](https://github.com/tonydavidx/flutter-android-alarm-manager-examples/tree/project-start).

open your `pubsepec.yml` file and paste the given android alarm manager package name under dependencies

```dart
android_alarm_manager: ^0.4.5+20
```

after that you need paste the following lines to your `AndroidManifest.xml`  file within the `<manifest></manifest>` tags 

```xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
<uses-permission android:name="android.permission.WAKE_LOCK"/>
```

Next add these lines inside application tag 

```xml
<service
    android:name="io.flutter.plugins.androidalarmmanager.AlarmService"
    android:permission="android.permission.BIND_JOB_SERVICE"
    android:exported="false"/>
<receiver
    android:name="io.flutter.plugins.androidalarmmanager.AlarmBroadcastReceiver"
    android:exported="false"/>
<receiver
    android:name="io.flutter.plugins.androidalarmmanager.RebootBroadcastReceiver"
    android:enabled="false">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"></action>
    </intent-filter>
</receiver>
```

now lets move on to next step. To work with alarm manager import android alarm manager inside `main.dart` file. 

```dart
import 'package:android_alarm_manager/android_alarm_manager.dart';
```

before setting any alarms first we need to Initialize android alarm manager service  we can do that in many ways one way of initializing android manager is calling `AndroidAlarmManager.initialize()` function inside apps main function. make sure it runs as an asynchronous function.

```dart
void main() async {
  runApp(MyApp());
  await AndroidAlarmManager.initialize();
}
```

restart your app if the alarm manager is initialized you will get `"Alarm service started"` message in the console. if you get any errors stop the app and run it again.

Inside switch button on changed method we are going to call the alarm manager timer functions. so when the switch turns on I want to set an alarm timer. there are three types of alarm timers `oneshot`, `oneshotAt`, `Periodic` I will explain how to use them and explain different between each timers. 

## `oneShot` Alarm Timer

 It takes three parameters first is delay we need to specify a Duration of delay for the callback function to start. then id will take a integer value it is used to identify the timer and we can cancel and replace existing timers. then callback it must be a top level function or a static method from a class. 

first for delay I am giving a 10 seconds. next create a integer for id, lets call it `alarmId` add 1 to it's value.  `int alarmID = 1;` for the callback create a top level function. a top level function are functions that defined outside class. lets call it fireAlarm 

```dart
void fireAlarm() {
  print('Alarm Fired at ${DateTime.now()}');
}
```

The print statement inside the curly braces will print current time when the alarm fires. 

```dart
class _HomePageState extends State<HomePage> {
  bool isOn = false;
// your code
  int alarmId = 1;
// your code
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Transform.scale(
          scale: 2,
          child: Switch(
            value: isOn,
            onChanged: (value) {
              setState(() {
                isOn = value;
              });
// your code
              AndroidAlarmManager.oneShot(
                Duration(seconds: 5),
                alarmId,
                fireAlarm,
              );
// your code
            },
          ),
        ),
      ),
    );
  }
}

void fireAlarm() {
  print('Alarm Fired at ${DateTime.now()}');
}
```

to test it turn on the switch button. if  everything goes correct you will see print statement of the `fireAlarm` function. along with the required parameters we can also add few more parameters, by setting `alarmClock` to `true` callback function will run as a Android Alarm. and if `wakeup` is set to `true` device will wake up when callback function runs. 

## Cancel Alarm Timers

after setting the alarm timers we need to cancel them as well. for that we can use `AndroidAlarmManager.cancel()` method. you need give a alarm id for this method to work.

```dart
Switch(
            value: isOn,
            onChanged: (value) {
              setState(() {
                isOn = value;
              });
              if (isOn == true) {
                AndroidAlarmManager.periodic(
                    Duration(seconds: 60), alarmId, fireAlarm);
              } else {
                AndroidAlarmManager.cancel(alarmId);
                print('Alarm Timer Canceled');
              }
            },
          ),
```

## oneShotAt Alarm Timer

this method works slightly different. `oneShotAt`  function schedules one shot timer to run call back at certain time so instead of delay duration it requires a Date and Time.  add a `DateTime()`  object inside parenthesis i am adding a time that is 30 seconds from now. you can give the values according to your time of practicing this tutorial.

`DateTime(2021,03,02,15,47,30)` 

you can keep other parameters as they are. 

```dart
class _HomePageState extends State<HomePage> {
  bool isOn = false;
  int alarmId = 1;
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Transform.scale(
          scale: 2,
          child: Switch(
            value: isOn,
            onChanged: (value) {
              setState(() {
                isOn = value;
              });
              if (isOn == true) {
// your code
                AndroidAlarmManager.oneShotAt(
                  DateTime(2021, 03, 02, 15, 47, 30),
                  alarmId,
                  fireAlarm,
// your code
                );
              } else {
                AndroidAlarmManager.cancel(alarmId);
                print('Alarm Timer Canceled');
              }
            },
          ),
        ),
      ),
    );
  }
}

void fireAlarm() {
  print('Alarm Fired at ${DateTime.now()}');
}
```

## Periodic Alarm Timer

Now lets see the third type of Timer function which is `Periodic` . it will take a Duration parameter and runs the callback function periodically for the given duration. so add a Duration object and give 60 seconds to its value. you cant run the method for less than 60 seconds interval. if you give a value of less than 60 seconds it will run the callback with 60 seconds interval.

```dart
class _HomePageState extends State<HomePage> {
  bool isOn = false;
  int alarmId = 1;
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Transform.scale(
          scale: 2,
          child: Switch(
            value: isOn,
            onChanged: (value) {
              setState(() {
                isOn = value;
              });
              if (isOn == true) {
                AndroidAlarmManager.periodic(
                    Duration(seconds: 60), alarmId, fireAlarm);
              } else {
                AndroidAlarmManager.cancel(alarmId);
                print('Alarm Timer Canceled');
              }
            },
          ),
        ),
      ),
    );
  }
}

void fireAlarm() {
  print('Alarm Fired at ${DateTime.now()}');
}
```

if you prefer to watch video tutorial, you can watch this tutorial as a video here   

[https://www.youtube.com/watch?v=F3qE6cLQk2I](https://www.youtube.com/watch?v=F3qE6cLQk2I)