import 'dart:io';

import 'package:intl/intl.dart';
import 'package:media_store_plus/media_store_plus.dart';
import 'package:path_provider/path_provider.dart';

import 'logger.dart';

class Utils {
  static String formatAmount(String value, {bool returnEmpty = false}) {
    var result = "";
    if (value.isNotEmpty&&value!="null") {
      //  NumberFormat numberFormat = NumberFormat.decimalPattern('hi');
      NumberFormat numberFormat = NumberFormat.decimalPattern('en_us');
      //  NumberFormat numberFormat = NumberFormat("###.0#", "en_US");
      // var formatterDecimal = NumberFormat('###.0#');
      result = numberFormat.format(double.parse(value));
    } else {
      result = "0";
    }
    if (returnEmpty && result == '0') {
      result = "";
    }
    return result;
  }

  static String formatDecimal(String? value, {bool returnEmpty = false}) {
    // var formatter = NumberFormat('#,##,000');
    var result = "";
    if (value != null &&value!="null"&&
        value.isNotEmpty &&
        value != "0.0000" &&
        value != "0" &&
        value != "0.0") {
      var formatterDecimal = NumberFormat('##0.00');
      result = formatterDecimal.format(double.parse(value));
    } else {
      result = "0.00";
    }
    if (returnEmpty && result == '0.00') {
      result = "";
    }
    return result;
  }
 static String formatDecimalThousand(String? value, {bool returnEmpty = false}) {
    // var formatter = NumberFormat('#,##,000');
    var result = "";
    if (value != null &&value!="null"&&
        value.isNotEmpty &&
        value != "0.0000" &&
        value != "0" &&
        value != "0.0") {
      NumberFormat formatter = NumberFormat.decimalPatternDigits(
        locale: 'en_us',
        decimalDigits: 4,
      );
      result = formatter.format(double.parse(value));
    } else {
      result = "0.00";
    }
    if (returnEmpty && result == '0.00') {
      result = "";
    }
    return result;
  }

  static String formatNumber(String? value, {bool returnEmpty = false}) {
    String result = "";

    if (value != null &&
        value.isNotEmpty &&
        value != "null" &&
        value != "0" &&
        value != "0.0" &&
        value != "0.00" &&
        value != "0.0000") {
      try {
        final double parsedValue = double.parse(value);
        final formatter = NumberFormat("#,##0.00", "en_US"); // comma + 2 decimals
        result = formatter.format(parsedValue);
      } catch (e) {
        result = "0.00"; // fallback in case parsing fails
      }
    } else {
      result = "0.00";
    }

    if (returnEmpty && result == '0.00') {
      result = "";
    }

    return result;
  }
  static String formatNumberUpTo4(String? value, {bool returnEmpty = false}) {
    String result = "";

    if (value != null &&
        value.isNotEmpty &&
        value != "null" &&
        value != "0" &&
        value != "0.0" &&
        value != "0.00" &&
        value != "0.0000") {
      try {
        final double parsedValue = double.parse(value);
        final formatter = NumberFormat("#,##0.0000", "en_US"); // comma + 2 decimals
        result = formatter.format(parsedValue);
      } catch (e) {
        result = "0.00"; // fallback in case parsing fails
      }
    } else {
      result = "0.00";
    }

    if (returnEmpty && result == '0.00') {
      result = "";
    }

    return result;
  }


  static num getDoubleFromString(String value) {
    String newValue = value.replaceAll(",", "");
    // NumberFormat format = NumberFormat.getInstance(Locale.getDefault());
    //Number number = format.parse(newvalue);
    return double.parse(newValue);
  }

  static String getCurrentDate() {
    DateFormat newDateFormat = DateFormat("yyyy-MM-dd");
    final now = new DateTime.now();
    String selectedDate = newDateFormat.format(now);
    return selectedDate;
  }

  static Future<bool> readOrWriteApiLevel33WithPermission(
      {required String initialRelativePath,
      required Function() operation}) async {
    final mediaStorePlugin = MediaStore();
    try {
      await operation();
      return true;
    } on FileSystemException catch (e) {

      // To test this place a .txt file in that folder
      // request for read/write access for a folder in API level 33
      // Use this also get write access for a folder from API level 30
      final documentTree = await mediaStorePlugin.requestForAccess(
          initialRelativePath: initialRelativePath);
      if (documentTree != null && documentTree.childrenUriList.isNotEmpty) {
        final uriString = documentTree.children
            .where((element) => element.name!.contains(".txt"))
            .first
            .uriString;
        documentTree.children.forEach((doc) {
        });

        // check if file exists by uri
        await mediaStorePlugin
            .isFileUriExist(uriString: uriString)
            .then((value) => value == true ? AppLogger.log("Exist") : AppLogger.log(""));

        // check if file is writable by uri
        await mediaStorePlugin
            .isFileWritable(uriString: uriString)
            .then((value) => value == true ? AppLogger.log("Writable") : AppLogger.log(""));

        // check if file is deletable by uri
        await mediaStorePlugin
            .isFileDeletable(uriString: uriString)
            .then((value) => value == true ? AppLogger.log("Deletable") : AppLogger.log(""));

        // read file by uri
        File tempFile = File(
            (await getApplicationSupportDirectory()).path + "/" + "text.txt");
        bool status = await mediaStorePlugin.readFileUsingUri(
            uriString: uriString, tempFilePath: tempFile.path);
        if (status) {
          AppLogger.log((await tempFile.readAsString()));
        }

        // edit file by uri
        await tempFile
            .writeAsString("EDITED: You are reading from API level 33.");
        await mediaStorePlugin.editFile(
            uriString: uriString, tempFilePath: tempFile.path);

        status = await mediaStorePlugin.readFileUsingUri(
            uriString: uriString, tempFilePath: tempFile.path);
        if (status) {
          AppLogger.log((await tempFile.readAsString()));
        }

        // delete file by uri
        // await mediaStorePlugin.deleteFileUsingUri(uriString: uriString);

        AppLogger.log("read folder info by folder uri");
        DocumentTree? docTree = await mediaStorePlugin.getDocumentTree(
            uriString: documentTree.uriString);
        if (docTree != null && docTree.childrenUriList.isNotEmpty) {
          final uriString = docTree.children
              .where((element) => element.name!.contains(".txt"))
              .first
              .uriString;
          AppLogger.log("2 Folder Uri ${docTree.uri.toString()}");
          AppLogger.log("2 File Uri ${uriString}");
          docTree.children.forEach((doc) {
            AppLogger.log("2 ${doc}");
          });

          // check if file exists by uri
          await mediaStorePlugin
              .isFileUriExist(uriString: uriString)
              .then((value) => value == true ? AppLogger.log("Exist") : AppLogger.log(""));

          // check if file is writable by uri
          await mediaStorePlugin
              .isFileWritable(uriString: uriString)
              .then((value) => value == true ? AppLogger.log("Writable") : AppLogger.log(""));

          // check if file is deletable by uri
          await mediaStorePlugin
              .isFileDeletable(uriString: uriString)
              .then((value) => value == true ? AppLogger.log("Deletable") : AppLogger.log(""));

          // read file by uri
          File tempFile = File(
              (await getApplicationSupportDirectory()).path + "/" + "text.txt");
          bool status = await mediaStorePlugin.readFileUsingUri(
              uriString: uriString, tempFilePath: tempFile.path);
          if (status) {
            AppLogger.log((await tempFile.readAsString()));
          }

          // edit file by uri
          await tempFile
              .writeAsString("EDITED: You are reading from API level 33.");
          await mediaStorePlugin.editFile(
              uriString: uriString, tempFilePath: tempFile.path);

          status = await mediaStorePlugin.readFileUsingUri(
              uriString: uriString, tempFilePath: tempFile.path);
          if (status) {
            AppLogger.log((await tempFile.readAsString()));
          }

          // delete file by uri
          // await mediaStorePlugin.deleteFileUsingUri(uriString: uriString);
        }
      }

      return true;
    } catch (e) {
      AppLogger.log(e);
      return false;
    }
    return false;
  }

  static File getFile({
    String? relativePath,
    required String fileName,
    required DirType dirType,
    required DirName dirName,
  }) {
    return File(
      dirType.fullPath(
              relativePath: relativePath.orAppFolder, dirName: dirName) +
          "/" +
          fileName,
    );
  }

  static String getPath({
    String? relativePath,
    required String fileName,
    required DirType dirType,
    required DirName dirName,
  }) {
    return dirType.fullPath(
            relativePath: relativePath.orAppFolder, dirName: dirName) +
        "/" +
        fileName;
  }
}
// To convert the string to pretty name
extension PrettyDocumentName on String {
  String toPrettyName() {
    if (isEmpty) return this;

    return replaceAll('_', ' ')                 // TARIFF SCHEDULE
        .split(' ')
        .map((word) => word.isEmpty
        ? ''
        : '${word[0].toUpperCase()}${word.substring(1).toLowerCase()}')
        .join(' ');
  }
}
