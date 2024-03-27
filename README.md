//  ------------------------------1--------------------------------------
 

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

  Future initLocalNotifications() async {
    const iOS = IOSInitializationSettings();
    const android = AndroidInitializationSettings('@drawable/ic_launcher');
    const settings = InitializationSettings(android: android, iOS: iOS);

    await _localNotification.initialize(
      settings,
      onSelectNotification: (payload) {
        final message =
            RemoteMessage.fromMap(jsonDecode(payload!) as Map<String, dynamic>);
        handelMessage(message);
      },
    );
    final platform = _localNotification.resolvePlatformSpecificImplementation<
        AndroidFlutterLocalNotificationsPlugin>();
    await platform?.createNotificationChannel(_adroidChannel);
  }

  Future<void> intiPushNotifications() async {
    // await FirebaseMessaging.instance
    //     .setForegroundNotificationPresentationOptions(
    //   alert: true,
    //   badge: true,
    //   sound: true,
    // );
    await _firebaseMessaging.setForegroundNotificationPresentationOptions(
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



//  ------------------------------2--------------------------------------


import 'dart:convert';
import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:get/get.dart';
import 'package:get/get_core/src/get_main.dart';
import 'package:mantisy_music_flutter/app/index.dart';

class FirebaseApi {
  final _firebaseMessaging = FirebaseMessaging.instance;

  final adroidChannel = const AndroidNotificationChannel(
    'high importance channel',
    'High Importance Notifications',
    description: 'This channel is used for importance notifications',
    importance: Importance.high,
  );
  final _localNotification = FlutterLocalNotificationsPlugin();

  void handleBackgroundMessage(RemoteMessage message) {
    print('Title: ${message.notification?.title}');
    print('Body: ${message.notification?.body}');
    print('Payload: ${message.data}');
  }

  void handleMessage(RemoteMessage message) {
    print('Handling message...');
    Get.toNamed(Routes.NOTIFICATION, arguments: message);

  }

  Future<void> initLocalNotifications() async {
    const iOS = IOSInitializationSettings();
    const android = AndroidInitializationSettings('@drawable/ic_launcher');
    const settings = InitializationSettings(android: android, iOS: iOS);

    await _localNotification.initialize(
      settings,
      onSelectNotification: (payload) {
        final message =
            RemoteMessage.fromMap(jsonDecode(payload!) as Map<String, dynamic>);
        handleMessage(message);
      },
    );
    final platform = _localNotification.resolvePlatformSpecificImplementation<
        AndroidFlutterLocalNotificationsPlugin>();
    await platform?.createNotificationChannel(adroidChannel);
  }

  Future<void> initPushNotifications() async {
    await _firebaseMessaging.requestPermission();
    final fCMToken = await _firebaseMessaging.getToken();
    print('Token : $fCMToken');

    await _firebaseMessaging.setForegroundNotificationPresentationOptions(
      alert: true,
      badge: true,
      sound: true,
    );

    FirebaseMessaging.instance.getInitialMessage().then((message) {
      if (message != null) {
        handleMessage(message);
      }
    });

    FirebaseMessaging.onMessageOpenedApp.listen(handleMessage);

    FirebaseMessaging.onBackgroundMessage((message) async {
      handleBackgroundMessage(message);
    });

    FirebaseMessaging.onMessage.listen((message) {
      final notification = message.notification;
      if (notification == null) return;
      _localNotification.show(
        notification.hashCode,
        notification.title,
        notification.body,
        NotificationDetails(
          android: AndroidNotificationDetails(
            adroidChannel.id,
            adroidChannel.name,
            channelDescription: adroidChannel.description,
            icon: '@drawable/ic_launcher',
            priority: Priority.high, // Set notification priority to high
          ),
        ),
        payload: jsonEncode(message.data),
      );
    });
  }

  Future<void> initNotifications() async {
    await initPushNotifications();
    await initLocalNotifications();
  }
}




//  ------------------------------3--------------------------------------



import 'dart:convert';

import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:get/get.dart';
import 'package:mantisy_music_flutter/app/index.dart';

class FirebaseApi {
  final adroidChannel = const AndroidNotificationChannel(
    'high importance channel',
    'High Importance Notifications',
    description: 'This channel is used for importance notifications',
    importance: Importance.high,
  );
  final flutterLocalNotificationsPlugin = FlutterLocalNotificationsPlugin();

  void handelMessage(RemoteMessage? message) {
    if (message == null) {
      return;
    }
    Get.toNamed(Routes.NOTIFICATION, arguments: message);
  }

  Future<void> handleBackgroundMessage(RemoteMessage message) async {
    print('Title: ${message.notification?.title}');
    print('Body: ${message.notification?.body}');
    print('Payload: ${message.data}');
  }

  Future initLocalNotifications() async {
    const iOS = IOSInitializationSettings();
    const android = AndroidInitializationSettings('@drawable/ic_launcher');
    const settings = InitializationSettings(android: android, iOS: iOS);

    await flutterLocalNotificationsPlugin.initialize(
      settings,
      onSelectNotification: (payload) async {
        final message =
            RemoteMessage.fromMap(jsonDecode(payload!) as Map<String, dynamic>);
        handelMessage(message);
      },
    );
  }

  Future<void> initNotifications() async {
    await FirebaseMessaging.instance.requestPermission();
    final fCMToken = await FirebaseMessaging.instance.getToken();

    await initLocalNotifications();

    FirebaseMessaging.onMessageOpenedApp.listen(handelMessage);

    FirebaseMessaging.onBackgroundMessage((message) async {
      handleBackgroundMessage(message);
    });

    FirebaseMessaging.onMessage.listen((message) {
      final notification = message.notification;
      if (notification == null) return;
      flutterLocalNotificationsPlugin.show(
        notification.hashCode,
        notification.title,
        notification.body,
        NotificationDetails(
          android: AndroidNotificationDetails(
            adroidChannel.id,
            adroidChannel.name,
            channelDescription: adroidChannel.description,
            icon: '@drawable/ic_launcher',
            priority: Priority.high, // Set notification priority to high
          ),
        ),
        payload: jsonEncode(message.toMap()),
      );
    });
  }
}
