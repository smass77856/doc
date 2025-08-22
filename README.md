# Tổng quan tệp đã chỉnh sửa

## Danh sách class đã sửa

- lib/config/store_secure_config.dart: `StoreSecureConfig`
- lib/config/store_setting.dart: `SharePreferenceKey`, `StoreConfig`, `StoreGlobalConfig`
- lib/global_constant.dart: `EnvConfig`
- lib/view_model/login_screen.viewmodel.dart: `LoginScreenViewModel`
- lib/ui/view/login-screen/login.screen.dart: `LoginScreen`, `_LoginScreenState`

## Các tệp Dart đã sửa trong thư mục `lib`

Các tệp sau đã được chỉnh sửa và cần được cập nhật trong source code:

### Tệp cấu hình

#### `lib/config/store_secure_config.dart`

- Thêm hỗ trợ lưu trữ sinh trắc học cho môi trường phát triển (development)
- Thay đổi chính:

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

- Chỉnh sửa phương thức sinh trắc học để hỗ trợ cả hai môi trường:

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

- Thêm các khóa dành cho môi trường phát triển:

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

- Cập nhật việc xóa dữ liệu cấu hình:

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

### Tệp lõi

#### `lib/global_constant.dart`

- Thêm kiểm tra môi trường phát triển:

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

- Cập nhật xử lý Firebase Messaging và khởi tạo ứng dụng

### View Model

#### `lib/view_model/login_screen.viewmodel.dart`

- Loại bỏ import không cần thiết và cập nhật xử lý đăng xuất:

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

### Màn hình UI

#### `lib/ui/view/login-screen/login.screen.dart`

-- Tái cấu trúc xác thực sinh trắc học:

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

- Thêm phương thức hỗ trợ mới:

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

