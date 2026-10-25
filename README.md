# تطبيقا طلبات صيانة الأدوات الكهربائية

مشروعان منفصلان بـ Flutter يشتركان في نفس قاعدة بيانات Firebase Firestore:

- **customer_app** — تطبيق العميل: يحدد موقعه تلقائيًا (GPS) ونوع العطل ويرسل الطلب.
- **provider_app** — تطبيقك أنت: يستقبل الطلبات لحظيًا، يعرضها، ويغيّر حالتها (قيد الانتظار / مقبول / مكتمل).

الربط بينهما تلقائي ومباشر عبر مجموعة Firestore واحدة اسمها `requests`، فأي طلب يُرسل من تطبيق العميل يظهر فورًا في تطبيقك.

---

## 1. إنشاء مشروع Firebase (مرة واحدة فقط)

1. اذهب إلى https://console.firebase.google.com وأنشئ مشروعًا جديدًا.
2. فعّل **Firestore Database** (ابدأ في وضع Test mode مبدئيًا، وعدّل القواعد لاحقًا — انظر قسم الأمان أسفله).
3. ثبّت الأدوات على جهازك:
   ```bash
   dart pub global activate flutterfire_cli
   npm install -g firebase-tools
   firebase login
   ```
4. داخل **كل** مجلد من المجلدين (`customer_app` و `provider_app`) نفّذ:
   ```bash
   flutterfire configure
   ```
   اختر نفس مشروع Firebase للتطبيقين (حتى يشتركا في نفس القاعدة)، وهذا سيولّد ملف `lib/firebase_options.dart` تلقائيًا في كل مجلد — هذا الملف غير موجود حاليًا في الكود، لازم تولّده بنفسك بهذه الخطوة.

## 2. هيكل بيانات الطلب (Firestore collection: `requests`)

```
requests/{requestId}
  customerName : string
  phone        : string
  faultType    : string   // "ثلاجة" | "غسالة" | "مكيف" | "سخان" | "أخرى"
  description  : string
  latitude     : number
  longitude    : number
  status       : string   // "pending" | "accepted" | "completed" | "rejected"
  createdAt    : timestamp
```

## 3. تشغيل كل تطبيق

```bash
cd customer_app
flutter pub get
flutter run          # للتجربة على جهاز متصل أو محاكي

cd ../provider_app
flutter pub get
flutter run
```

## 4. بناء ملف APK حقيقي

```bash
flutter build apk --release
```
الملف الناتج يكون في: `build/app/outputs/flutter-apk/app-release.apk`

نفّذ هذا الأمر داخل كل مجلد على حدة (مرة لـ customer_app ومرة لـ provider_app) — تحصل على ملفين APK منفصلين.

## 5. صلاحيات الموقع (GPS)

ملف `android/app/src/main/AndroidManifest.xml` في **customer_app** يحتاج هذين السطرين داخل tag `<manifest>` (قبل `<application>`):
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

## 6. قواعد أمان Firestore (مهم قبل الإطلاق الفعلي)

وضع الاختبار (Test mode) يسمح لأي شخص بالقراءة والكتابة — غير آمن للإنتاج. مثال مبسّط لقواعد أكثر أمانًا (تتطلب لاحقًا ربط Firebase Authentication لتمييز العميل عن مزود الخدمة):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /requests/{requestId} {
      allow create: if true;           // أي عميل يقدر ينشئ طلب
      allow read, update: if true;     // بدّلها لاحقًا لتشترط تسجيل دخول
    }
  }
}
```
هذا مبسّط للتجربة فقط — قبل النشر الفعلي للمستخدمين لازم تضيف Firebase Authentication وتحصر الكتابة/القراءة حسب هوية المستخدم.

## 7. التوسعات المقترحة لاحقًا
- إشعارات فورية (Firebase Cloud Messaging) عند وصول طلب جديد لتطبيقك، أو عند تغيّر حالة الطلب للعميل.
- تسجيل دخول العملاء (Firebase Auth) لربط كل طلب بحساب.
- عرض الموقع على خريطة فعلية (google_maps_flutter) بدل رابط خارجي فقط.
