# Modified Files Summary

## Modified Dart Files in `lib` folder

The following files have been modified and need to be updated in the source code:

### Config Files

#### `lib/config/store_secure_config.dart`

- Added support for development environment biometric storage
- Key changes:

```diff
-final storage = FlutterSecureStorage();
+var storage = FlutterSecureStorage();
+FlutterSecureStorage secureStorage = const FlutterSecureStorage();

 class StoreSecureConfig {
+  static String emailBio = "email_bio";
+  static String passBio = "pass_bio";
+  static String emailBioDev = "email_bio_dev";
+  static String passBioDev = "pass_bio_dev";

+  static String getEmailBio() {
+    if (EnvConfig.checkIsDev) {
+      return emailBioDev;
+    } else {
+      return emailBio;
+    }
  }

+  static void removeAllDataLocalStorage() {
+    if (EnvConfig.checkIsDev) {
+      storage.delete(key: emailBioDev);
+      storage.delete(key: passBioDev);
+      storage.delete(key: SharePreferenceKey.allowBiometricDev);
+      storage.delete(key: SharePreferenceKey.isUsingBiometricDev);
+    } else {
+      storage.delete(key: emailBio);
+      storage.delete(key: passBio);
+      storage.delete(key: SharePreferenceKey.allowBiometric);
+      storage.delete(key: SharePreferenceKey.isUsingBiometric);
+    }
+  }
```

- Modified biometric methods to support both environments:

```diff
 static Future<void> storeValueBioMetric(String value) async {
-  await storage.write(
-      key: SharePreferenceKey.allowBiometric,
-      value: value,
-      iOptions: iosOptions);
+  if (EnvConfig.checkIsDev) {
+    await storage.write(
+        key: SharePreferenceKey.allowBiometricDev,
+        value: value,
+        iOptions: iosOptions);
+  } else {
+    await storage.write(
+        key: SharePreferenceKey.allowBiometric,
+        value: value,
+        iOptions: iosOptions);
+  }
}
```

#### `lib/config/store_setting.dart`

- Added development environment keys:

```diff
class SharePreferenceKey {
+  //prod
   static String isFirstLogin = "isFirstLogin";
   // ... existing keys ...
+
+  //dev
+  static String isFirstLoginDev = "isFirstLoginDev";
+  static String isFirstConsentDev = "isFirstConsentDev";
+  static String storeFeatureKeyDev = "storeFunctionKeyDev";
+  static String languageKeyDev = "languageKeyDev";
+  static String authenBioMetricDev = "authenBioMetricDev";
+  static String hostNameAdminDev = "hostNameAdminDev";
+  // ... more dev keys ...
}
```

- Updated config store clearing:

```diff
static Future<void> clearConfigStore() async {
  pref = await SharedPreferences.getInstance();
-  await pref!.remove(SharePreferenceKey.languageKey);
-  await pref!.remove(SharePreferenceKey.storeFeatureKey);
-  await pref!.remove(SharePreferenceKey.authenBioMetric);
-  await pref!.remove(SharePreferenceKey.hostNameAdmin);
-  await pref!.remove(SharePreferenceKey.fcmToken);
-  await pref!.remove(SharePreferenceKey.allowBiometric);
+  if (EnvConfig.checkIsDev) {
+    await pref!.remove(SharePreferenceKey.languageKeyDev);
+    await pref!.remove(SharePreferenceKey.storeFeatureKeyDev);
+    await pref!.remove(SharePreferenceKey.authenBioMetricDev);
+    await pref!.remove(SharePreferenceKey.hostNameAdminDev);
+    await pref!.remove(SharePreferenceKey.fcmTokenDev);
+    await pref!.remove(SharePreferenceKey.allowBiometricDev);
+  } else {
+    await pref!.remove(SharePreferenceKey.languageKey);
+    await pref!.remove(SharePreferenceKey.storeFeatureKey);
+    await pref!.remove(SharePreferenceKey.authenBioMetric);
+    await pref!.remove(SharePreferenceKey.hostNameAdmin);
+    await pref!.remove(SharePreferenceKey.fcmToken);
+    await pref!.remove(SharePreferenceKey.allowBiometric);
+  }
}
```

### Core Files

#### `lib/global_constant.dart`

- Added development environment check:

```diff
class EnvConfig {
  static final String urlApi = dotenv.env['URL_API']!;
  static final String auth0Domain = dotenv.env['AUTH0_DOMAIN']!;
  static final String auth0ClientId = dotenv.env['AUTH0_CLIENT_ID']!;
  static final String auth0ClientSecret = dotenv.env['AUTH0_CLIENT_SECRET']!;
  static final String auth0RedirectUri = dotenv.env['AUTH0_REDIRECT_URI']!;
  static final String auth0Issuer = dotenv.env['AUTH0_ISSUER']!;
+ static final String isDev = dotenv.env['IS_DEV']!;

+ static bool get checkIsDev => isDev == "true";
}
```

#### `lib/main.dart`

- Updated Firebase messaging handling and app initialization

### View Models

#### `lib/view_model/login_screen.viewmodel.dart`

- Removed unnecessary imports and updated logout handling:

```diff
-import 'package:flutter_secure_storage/flutter_secure_storage.dart';
 // ...
-import 'package:vegas_club/service/common.service.dart';

 FlutterAppAuth? appAuth = const FlutterAppAuth();
-FlutterSecureStorage secureStorage = const FlutterSecureStorage();
```

```diff
 // In logout method:
 await StoreConfig.clearConfigStore();
 await StoreSecureConfig.storeValueIsUsingBioMetric("false");
-await StoreGlobalConfig.setConfigToStore(
-    SharePreferenceKey.isUsingBiometric, false);
-await boxAuth!.clear();
-await secureStorage.deleteAll();
+if (EnvConfig.checkIsDev) {
+  await StoreGlobalConfig.setConfigToStore(
+      SharePreferenceKey.isUsingBiometricDev, false);
+} else {
+  await StoreGlobalConfig.setConfigToStore(
+      SharePreferenceKey.isUsingBiometric, false);
+}
+// await boxAuth!.clear();
+StoreSecureConfig.removeAllDataLocalStorage();
```

### UI Screens

#### `lib/ui/view/login-screen/login.screen.dart`

- Refactored biometric authentication:

```diff
Future<void> _authenticateWithBiometrics() async {
-  bool authenticated = false;
-
-  var value = await StoreSecureConfig.getValueIsUsingBioMetric();
-
-  var emailBiometric =
-      await storage.read(key: "email_bio", aOptions: _getAndroidOptions());
-  var passBiometric =
-      await storage.read(key: "pass_bio", aOptions: _getAndroidOptions());
-  if (emailBiometric == null || passBiometric == null) {
-    await StoreSecureConfig.removeValueBioMetric();
-
-    return;
-  }
-  var valueAllowbiometric = await StoreSecureConfig.getValueBioMetric();
-  if (valueAllowbiometric == null ||
-      valueAllowbiometric == "false" ||
-      valueAllowbiometric != "true") {
-    await StoreSecureConfig.removeValueBioMetric();
-    return;
-  }
-  // ...
+  try {
+    // Đọc thông tin đăng nhập từ storage một lần
+    final emailBiometric = await storage.read(
+        key: StoreSecureConfig.getEmailBio(), aOptions: _getAndroidOptions());
+    final passBiometric = await storage.read(
+        key: StoreSecureConfig.getPassBio(), aOptions: _getAndroidOptions());
+    final valueAllowbiometric = await StoreSecureConfig.getValueBioMetric();
+    final value = await StoreSecureConfig.getValueIsUsingBioMetric();
+
+    // Kiểm tra điều kiện một lần
+    if (emailBiometric == null ||
+        passBiometric == null ||
+        valueAllowbiometric != "true" ||
+        value != "true") {
+      await StoreSecureConfig.removeValueBioMetric();
+      return;
+    }
```

- Added new helper method:

```diff
+  // Hàm xử lý đăng nhập thành công
+  Future<void> _handleSuccessfulLogin(int customerId) async {
+    locator.get<MixPanelTrackingService>().initMixpanel(callBack: () {
+      locator.get<MixPanelTrackingService>().identity();
+    });

+    await storage.write(
+        key: StorageEnum.loginSucessByFaceId.name, value: "true");
+    await getTokenDevice();

+    SettingSystemConfig().initSetting(context, onSuccess: () {
+      Provider.of<ApplicationViewmodel>(context, listen: false)
+          .getSettingValueNotificationType();
+    });

+    if (customerId == 0) {
+      Navigator.of(context).pushReplacementNamed(AdminHomeScreen.routeName);
+    } else {
+      Navigator.of(context).pushReplacementNamed(HomeScreen.route);
+    }
+  }
```

## Note

These files have been modified in the `phase_dev_release_06_2025` branch. Please ensure these changes are properly integrated into your codebase.
