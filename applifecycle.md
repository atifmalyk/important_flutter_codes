import 'package:flutter/material.dart';
import 'package:get/get.dart';
import 'package:marlin_app/app/modules/root/controllers/root_controller.dart';
import 'package:marlin_app/app_components/MyApplication.dart';

import '../../utils/logger.dart';

class AppLifecycleReactor with WidgetsBindingObserver {
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    // AppLogger.log("$state");

    if (state == AppLifecycleState.resumed &&
        Get.isRegistered<RootController>()) {
      // App moved from background → foreground
      RootController controller = Get.find();
      if (MyApplication.appController.isFromShare.value == false) {
        controller.checkIsUserLoggedIn();
      }
      MyApplication.appController.isFromShare.value = false;
    }

    if (state == AppLifecycleState.paused &&
        Get.isRegistered<RootController>()) {
      MyApplication.appController.disconnectSocket();
      MyApplication.appController.isFromShare.value = false;
    }
    if (state == AppLifecycleState.inactive &&
        MyApplication.appController.isFromShare.value == false) {
      MyApplication.appController.disconnectSocket();
    }
  }
}
