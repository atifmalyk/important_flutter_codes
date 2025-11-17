import 'dart:io';
import 'package:flutter/foundation.dart';
import 'package:safe_device/safe_device.dart';
import 'package:safe_device/safe_device_config.dart';

class SecurityService {
  /// Initialize safe_device with comprehensive checks
  static Future<void> initialize() async {
    if (!kIsWeb) {
      // SafeDevice.init() returns void, so no await needed
      // Default config enables all checks (mock location, root, jailbreak)
      SafeDevice.init(
        SafeDeviceConfig(), // Default config enables all standard checks
      );
    }
  }

  /// Check if device is rooted (Android) or jailbroken (iOS)
  // static Future<bool> get isDeviceRootedOrJailbroken async {
  //   if (kIsWeb) return false;
  //
  //   try {
  //     return await SafeDevice.isJailBroken;
  //   } catch (e) {
  //     debugPrint('Root/Jailbreak detection error: $e');
  //     return false; // Assume safe if detection fails
  //   }
  // }
  //
  // /// Check if running on emulator/simulator
  // static Future<bool> get isEmulator async {
  //   if (kIsWeb) return false;
  //
  //   try {
  //     // Use safe_device's built-in real device check first (more reliable)
  //     final isRealDevice = await SafeDevice.isRealDevice;
  //     if (!isRealDevice) return true;
  //
  //     // Fallback to device_info_plus for additional emulator detection
  //     final deviceInfo = DeviceInfoPlugin();
  //
  //     if (Platform.isAndroid) {
  //       final androidInfo = await deviceInfo.androidInfo;
  //       return !androidInfo.isPhysicalDevice || _isAndroidEmulator(androidInfo);
  //     } else if (Platform.isIOS) {
  //       final iosInfo = await deviceInfo.iosInfo;
  //       return !iosInfo.isPhysicalDevice;
  //     }
  //   } catch (e) {
  //     debugPrint('Emulator detection error: $e');
  //   }
  //
  //   return false;
  // }
  //
  // static bool _isAndroidEmulator(AndroidDeviceInfo androidInfo) {
  //   final model = androidInfo.model?.toLowerCase() ?? '';
  //   final product = androidInfo.product?.toLowerCase() ?? '';
  //   final fingerprint = androidInfo.fingerprint?.toLowerCase() ?? '';
  //   final brand = androidInfo.brand?.toLowerCase() ?? '';
  //   final device = androidInfo.device?.toLowerCase() ?? '';
  //   final hardware = androidInfo.hardware?.toLowerCase() ?? '';
  //   final manufacturer = androidInfo.manufacturer?.toLowerCase() ?? '';
  //
  //   return model.contains('sdk') ||
  //       product.contains('emulator') ||
  //       fingerprint.contains('test-keys') ||
  //       brand.contains('generic') ||
  //       device.contains('generic') ||
  //       hardware.contains('goldfish') ||
  //       hardware.contains('ranchu') ||
  //       manufacturer.contains('genymotion') ||
  //       product.contains('sdk_gphone');
  // }
  //
  // /// Check developer mode (Android only) + iOS simulator debug
  // static Future<bool> get isDeveloperMode async {
  //   if (kIsWeb) return false;
  //
  //   try {
  //     if (Platform.isAndroid) {
  //       return await SafeDevice.isDevelopmentModeEnable;
  //     } else if (Platform.isIOS) {
  //       // iOS doesn't have direct developer mode, but check simulator
  //       return !(await SafeDevice.isRealDevice);
  //     }
  //     return false;
  //   } catch (e) {
  //     debugPrint('Developer mode detection error: $e');
  //     return false;
  //   }
  // }
  //
  // /// Check for mock location (Android only)
  // static Future<bool> get isMockLocationEnabled async {
  //   if (!Platform.isAndroid || kIsWeb) return false;
  //
  //   try {
  //     return await SafeDevice.isMockLocation;
  //   } catch (e) {
  //     debugPrint('Mock location detection error: $e');
  //     return false;
  //   }
  // }
  //
  // /// Check if app is running in debug mode
  // static Future<bool> get isDebugBuild async {
  //   try {
  //     return await SafeDevice.isUsbDebuggingEnabled;
  //   } catch (e) {
  //     return kDebugMode; // Fallback to Flutter's debug mode
  //   }
  // }

  /// Comprehensive security check with detailed reporting
  static Future<SecurityCheckResult> performFullSecurityCheck() async {
    try {
      final checks = await Future.wait([
        //SafeDevice.isSafeDevice,
        //SafeDevice.isRealDevice,
        // SafeDevice.isJailBroken.then((v) => !v),
      //  SafeDevice.isMockLocation.then((v) => !v),
      //  SafeDevice.isDevelopmentModeEnable.then((v) => !v),
     //   SafeDevice.isUsbDebuggingEnabled.then((v) => !v),
      ]);

      final isSecure = checks.every((check) => check == true);

      return SecurityCheckResult(
        isSecure: isSecure,
        details: {
          // 'isSafeDevice': await SafeDevice.isSafeDevice,
          'isJailBroken': await SafeDevice.isJailBroken,
          // 'isRealDevice': await SafeDevice.isRealDevice,
          // 'isMockLocation': await SafeDevice.isMockLocation,
          'isDevelopmentMode': await SafeDevice.isDevelopmentModeEnable,
          // 'isDebuggable': await SafeDevice.isUsbDebuggingEnabled,
          'jailbreakDetails': await SafeDevice.jailbreakDetails,
          'rootDetails': await SafeDevice.rootDetectionDetails,
        },
      );
    } catch (e) {
      debugPrint('Full security check failed: $e');
      return SecurityCheckResult(
        isSecure: false,
        details: {'error': e.toString()},
      );
    }
  }

  /// Get detailed security violation message
  static Future<String?> getDetailedSecurityMessage() async {
    final result = await performFullSecurityCheck();

    if (!result.isSecure) {
      final details = result.details;

      if (details['isJailBroken'] == true) {
        return Platform.isAndroid
            ? "This device is rooted. The app cannot run on rooted devices for security reasons."
            : "This device is jailbroken. The app cannot run on jailbroken devices for security reasons.";
      }

      if (details['isRealDevice'] == false) {
        return "This app cannot be run on emulators or simulators for security reasons.";
      }

      if (details['isMockLocation'] == true) {
        return "Mock location is enabled. Please disable fake GPS apps and developer mock location settings.";
      }

      if (details['isDevelopmentMode'] == true) {
        return Platform.isAndroid
            ? "Developer options are enabled. Please disable developer mode to continue."
            : "This device appears to be in development mode.";
      }

      if (details['isDebuggable'] == true) {
        return "Debug mode detected. Please use a release build.";
      }

      return "Device security check failed. Please ensure your device is secure and unmodified.";
    }

    return null;
  }
}

class SecurityCheckResult {
  final bool isSecure;
  final Map<String, dynamic> details;

  SecurityCheckResult({
    required this.isSecure,
    required this.details,
  });
}
