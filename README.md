// ignore_for_file: strict_raw_type

import 'dart:convert';
import 'dart:io';

import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:get/get.dart';
import 'package:get/get_core/src/get_main.dart';
import 'package:mantisy_music_flutter/app/index.dart';
import 'package:mantisy_music_flutter/app/modules/notification/notification_page.dart';

Future<void> handleBackgroundMessage(RemoteMessage message) async {
  print('Title: ${message.notification?.title}');
  print('Body: ${message.notification?.body}');
  print('Payload: ${message.data}');
}

class FirebaseApi {
  final _firebaseMessaging = FirebaseMessaging.instance;

  final _adroidChannel = AndroidNotificationChannel(
    'high imprtance channel',
    'High Importance Notifications',
    description: 'This channel is used for imprtance notification ',
    importance: Importance.defaultImportance,
  );
  final _localNotification = FlutterLocalNotificationsPlugin();

  void handelMessage(RemoteMessage? message) {
    print('dsfkljsdkfhjks');
    if (message == null) {
      return;
    }

    // navigatorKey.currentState?.pushNamed(
    //   NotificationPage.route,
    //   arguments: message,
    // );

    Get.toNamed(Routes.NOTIFICATION, arguments: message);
  }

  // Future initLocalNotifications() async {
  //   const iOS = IOSInitializationSettings();
  //   const android = AndroidInitializationSettings('@drawable/ic_launcher');
  //   const settings = InitializationSettings(android: android, iOS: iOS);

  //   await _localNotification.initialize(
  //     settings,
  //     onSelectNotification: (payload) {
  //       final message =
  //           RemoteMessage.fromMap(jsonDecode(payload!) as Map<String, dynamic>);
  //       handelMessage(message);
  //     },
  //   );
  //   final platform = _localNotification.resolvePlatformSpecificImplementation<
  //       AndroidFlutterLocalNotificationsPlugin>();
  //   await platform?.createNotificationChannel(_adroidChannel);

  // }

  Future<void> initLocalNotifications() async {
    const iOS = IOSInitializationSettings();
    const android = AndroidInitializationSettings('@drawable/ic_launcher');
    const settings = InitializationSettings(android: android, iOS: iOS);

    final flutterLocalNotificationsPlugin = FlutterLocalNotificationsPlugin();
    await flutterLocalNotificationsPlugin.initialize(
      settings,
      onSelectNotification: (payload) async {
        final message =
            RemoteMessage.fromMap(jsonDecode(payload!) as Map<String, dynamic>);
        handelMessage(message);
      },
    );

    final platform =
        flutterLocalNotificationsPlugin.resolvePlatformSpecificImplementation<
            AndroidFlutterLocalNotificationsPlugin>();
    await platform?.createNotificationChannel(_adroidChannel);

    // Show a notification as a snack bar
    flutterLocalNotificationsPlugin.show(
      0,
      'Notification Title',
      'Notification Body',
      NotificationDetails(
        android: AndroidNotificationDetails(
          'your_channel_id',
          'your_channel_name',
          // 'your_channel_description',
          importance: Importance.max,
          priority: Priority.high,
        ),
      ),
    );
  }













  Future<void> intiPushNotifications() async {
    await FirebaseMessaging.instance
        .setForegroundNotificationPresentationOptions(
      alert: true,
      badge: true,
      sound: true,
    );
    FirebaseMessaging.instance.getInitialMessage().then(handelMessage);
    FirebaseMessaging.onMessageOpenedApp.listen(handelMessage);
    FirebaseMessaging.onBackgroundMessage(handleBackgroundMessage);

    FirebaseMessaging.onMessage.listen((message) {
      final notification = message.notification;
      if (notification == null) return;
      _localNotification.show(
        notification.hashCode,
        notification.title,
        notification.body,
        NotificationDetails(
          android: AndroidNotificationDetails(
            _adroidChannel.id,
            _adroidChannel.name,
            channelDescription: _adroidChannel.description,
            icon: '@drawable/ic_launcher',
          ),
        ),
        payload: jsonEncode(message.toMap()),
      );
    });
  }

  Future<void> initNotifications() async {
    await _firebaseMessaging.requestPermission();
    final fCMToken = await _firebaseMessaging.getToken();
    print('Token : $fCMToken');
    print('dsfjhsdkjfhskdj');
    intiPushNotifications();
    initLocalNotifications();
  }
}

