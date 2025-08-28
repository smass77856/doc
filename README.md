# Tổng quan tệp đã chỉnh sửa (có thể vào link dưới để đọc cho dễ)
## Hiện tại do xung đột môi trường DEV và PROD trên thiết bị nên bị lỗi FACEID và phần lưu trạng thái của toggle FaceID trong setting.
https://github.com/smass77856/doc

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

## Ghi chú

Các tệp này đã được chỉnh sửa trong nhánh `phase_dev_release_06_2025`. Vui lòng đảm bảo những thay đổi này được tích hợp đầy đủ vào mã nguồn của bạn.

---

## High light thay đổi cho UI (để DEV tích hợp nhanh)

### `lib/ui/view/more-screen/more.screen.dart`

- Các điểm thay đổi chính liên quan Face ID/Touch ID và lưu trạng thái:

```diff
@@ class _MoreScreenState extends StateConsumer<MoreScreen> with BaseFunction {
   bool? _canCheckBiometrics;
   _SupportState _supportState = _SupportState.unknown;
   List<BiometricType>? _availableBiometrics;
   bool _isEnabled = false;
   String? languageCode;
   final RefreshController _refreshController = RefreshController();
   final LocalAuthentication auth = LocalAuthentication();
@@
   _isEnabled = await StoreSecureConfig.getValueBioMetric() == "true";
   auth.isDeviceSupported().then(
         (bool isSupported) => setState(() => _supportState = isSupported
             ? _SupportState.supported
             : _SupportState.unsupported),
       );
   _checkBiometrics();
   _getAvailableBiometrics();
@@
   void toggleSwitch(bool value) {
     if (_availableBiometrics != null && _availableBiometrics!.isNotEmpty) {
       if (value == false) {
         // show confirm dialog
         // on OK:
+        StoreSecureConfig.storeValueBioMetric("false");
         setState(() { _isEnabled = false; });
       } else {
         if (EnvConfig.checkIsDev) {
           StoreGlobalConfig.setConfigToStore<bool>(SharePreferenceKey.isUsingBiometricDev, true);
           StoreGlobalConfig.setConfigToStore<bool>(SharePreferenceKey.allowBiometricDev, true);
         } else {
           StoreGlobalConfig.setConfigToStore<bool>(SharePreferenceKey.isUsingBiometric, true);
           StoreGlobalConfig.setConfigToStore<bool>(SharePreferenceKey.allowBiometric, true);
         }
+        StoreSecureConfig.storeValueBioMetric("true");
+        StoreSecureConfig.storeValueIsUsingBioMetric("true");
         setState(() { _isEnabled = true; });
       }
     } else {
       // show guide dialog to enable biometric in device settings
     }
   }
```

- Tóm tắt:
  - Đọc trạng thái bật/tắt từ `StoreSecureConfig.getValueBioMetric()` thay vì chỉ `StoreGlobalConfig`.
  - Ghi trạng thái cho phép vào SecureStorage: `storeValueBioMetric("true"|"false")`, `storeValueIsUsingBioMetric("true")` khi bật.
  - Giữ phân tách DEV/PROD bằng `EnvConfig.checkIsDev` đối với `StoreGlobalConfig`.

### `lib/ui/view/signin-screen/sign_in.screen.dart`

- Các điểm thay đổi chính để đăng nhập bằng sinh trắc học khớp storage mới:

```diff
@@ class _SignInScreenState extends State<SignInScreen> with BaseFunction {
   final LocalAuthentication auth = LocalAuthentication();
   _SupportState _supportState = _SupportState.unknown;
   bool? _canCheckBiometrics;
   List<BiometricType>? _availableBiometrics;
@@
   Future<void> _getEmailBio() async {
     var value = await storage.read(
         key: StoreSecureConfig.getEmailBio(), aOptions: _getAndroidOptions());
     setState(() { _emailBio = value ?? ""; });
   }

   Future<void> _getPassBio() async {
     var value = await storage.read(
         key: StoreSecureConfig.getPassBio(), aOptions: _getAndroidOptions());
     setState(() { _passBio = value ?? ""; });
   }
@@ // Sau khi login thường thành công, ghi lại thông tin để lần sau dùng Face/Touch ID
   await storage.write(
       key: StoreSecureConfig.getEmailBio(),
       value: _emailController.text,
       aOptions: _getAndroidOptions());
   await storage.write(
       key: StoreSecureConfig.getPassBio(),
       value: _passwordController.text,
       aOptions: _getAndroidOptions());
@@ // Nút Face/Touch ID
   if (_supportState == _SupportState.supported)
     GestureDetector(
       onTap: () {
         if (isAllowbiometric != true && _availableBiometrics != null && _availableBiometrics!.isEmpty) {
           return;
         }
         if (isAllowbiometric == true && _availableBiometrics != null && _availableBiometrics!.isEmpty) {
           // dialog hướng dẫn vào cài đặt thiết bị
         }
         if (isAllowbiometric == true && _availableBiometrics != null && _availableBiometrics!.isNotEmpty) {
           _authenticateWithBiometrics();
         }
         if (isAllowbiometric != true && _availableBiometrics != null && _availableBiometrics!.isNotEmpty) {
           // dialog hướng dẫn bật trong app setting
         }
       },
       child: // icon Face/Touch
     );
@@ // Đăng nhập sinh trắc học (điều kiện + cờ sử dụng)
   Future<void> _authenticateWithBiometrics() async {
     bool authenticated = false;
     try {
       setState(() { _isAuthenticating = true; _authorized = 'Authenticating'; });
       authenticated = await auth.authenticate(
         localizedReason: 'Scan your fingerprint (or face or whatever) to authenticate',
         options: const AuthenticationOptions(stickyAuth: true, biometricOnly: true),
       );
       setState(() { _isAuthenticating = false; _authorized = 'Authenticating'; });
     } on PlatformException catch (e) {
       setState(() { _isAuthenticating = false; _authorized = 'Error - ${e.message}'; });
       return;
     }
     if (!mounted) { return; }
     if (authenticated) {
+      await StoreSecureConfig.storeValueIsUsingBioMetric("true");
       await StoreGlobalConfig.setConfigToStore(SharePreferenceKey.isUsingBiometric, true);
       // gọi eventSignin với _emailBio/_passBio
     }
   }
```

- Tóm tắt:
  - Đọc/ghi email & password cho Face/Touch ID bằng `StoreSecureConfig.getEmailBio()/getPassBio()`.
  - Sau khi đăng nhập bằng sinh trắc học thành công, cập nhật cờ sử dụng: `storeValueIsUsingBioMetric("true")` và `isUsingBiometric` trong `StoreGlobalConfig`.
  - Điều kiện hiển thị/khả dụng của nút Face/Touch ID phụ thuộc `isAllowbiometric` và danh sách `getAvailableBiometrics()`.

