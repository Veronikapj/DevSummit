# Migrate your apps to Android13

https://youtu.be/wBx3-ZObxY8

## Migration Plan
- Behavior changes
- New capabilites

## Behavior changes
behavior
- System Health   
    Regarding both system performance and bettery life
- Privacy   
    Minimize your data sharing when ever possible

versions
- All Apps   
    Applies to all apps on the platform, regardless of which SDK version is targeted
- Target SDK Version  
    Only applies if the app targets a specific SDK version (or higher)

## Test Your App
Against the new release
- Android Emulator
- Supported Pixel devices
- Select Android partner devices
- Firebase Test Lab

# Impacting all apps
(regardless of targetSdkVersion)  
사용자에게 앱의  백그라운드 작업에  대해  알리고  제어권을 제공한다.
- Related to system health 
- Dismissable  
    Foregroud services nofications (Foreground services 알림 닫는 기능 추가)
-----
## Foreground Services (all)

Task Manager   
앱이 포그라운드 서비스를 사용하고 있을 경우, 이는 새로운 인터페이스에 포함되고 사용자가 포그라운드 서비스를 종료할 수 있습니다. 

Stopping you foreground services using ADB 

```
adb shell cmd activity stop-app PACKAGE_NAME 
```

`stop-app` 명령을 사용해서 앱이 포그라운드 서비스 작업 관리자에서 앱을 정지할때와 동일하게 테스트 할 수 있습니다. 이 방법은 여러가지 면에서 force-stop 명령과 유사합니다. 

```
adb shell am kill PACKAGE_NAME
adb shell cmd activity stop-app PACKAGE_NAME
adb shell cmd activity force-stop PACKAGE_NAME
```

![Migrate your apps to Android 13 3-21 screenshot](https://user-images.githubusercontent.com/360685/208286732-bbcaa93e-e4b1-4ca6-b85e-52a08e3e00b8.png)

force-stop 과 달리 기록에서 앱이 제거되지 않고 예약된 작업과 알람이 취소되지 않습니다. 

### Getting ApplicationExitInfo

포그라운드 작업 관리자에서 앱을 정지하면 시스템 콜백은 받지 않지만, 다음번에 ApplicationExitInfo Api 를 통해 앱이 시작할 때 결과를 확인  
사용자가 앱을 정지 했는 지, 오류, ANR, 처리되지 않은 예외 메모리 부족 등의 문제 확인 

```kotlin
val am = getSystemService<ActivityManager>()
am?.let {
    val exitReasons = it.getHistoricalProcessExitReasons(null, 0, 1)
    if (exitReasons.isNotEmpty()) {
        val exitInfo = exitReasons.first()
        val reason = exitInfo.reason // user affordance, will be REASON_USER_REQUESTED
    }
}
```

## FGS Test Manager
- Data Robustness   
    Minimize data loss when the app process is killed (앱이 어떤 상황에서도 정지 명령 처리)  
- Foreground Services  
    Eliminate unnecessary forground services (불필요한 포그라운드 서비스 제거)  

-----------
## Restricted Changes (all)
Your app is much more likely to end up in the restricted app standby bucket in Android 13  
백그라운드 액티비티일 경우 android 13에서는 앱에서 활동이 없어서 제한된 앱 대기 버킷으로 대체되기까지의 시간이 훨씬 감소 
- 45 Days : Android 12, 12L
- **8 Days** : Android 13

### Restricted
App Standby Bucket
- Limited to running jobs once per day in a 10 minute batched session 
- Run fewer expedited jobs
- Invoke one alarm per day 

#### Detecting restricted bucket
앱이 제한된 모든임을 감지하면 적절한 필수작업을 예약할 수 있도록 지원 가능   
앱이 실행중인 상태에서는 테스트 불가능 : 앱이 상호작용 중인 상탤르 인식하고 제한된 버킷에서 빠져나간다. 
```kotlin
val usageStatsManager = getSystemService<UsageStatsManager>()
usageStatsManager?.let {
    if (it.appStandbyBucket == STANDBY_BUCKET_RESTRECTED) {
        . . . // 작업과 알림 실행 할 때 앱 상태를 로깅 하는 것이 좋음 
    }
}
```

제한된 버킷으로 들어가지 않게 하려면?  
사용자가 해당 앱에 대한 실행 가능 알림을 받도록 사용자 동의가 필요   
- ~before SDK 33 : Android가 개발자 대신 권한을 요청   
- SDK 33 after : 사용자에게 알림 권한을 직접 요청 가능 - 권한 대화 상자를 띄우는 시점을 완벽히 제어

### Confirming enabled notifications
```kotlin
val noficiationsManagerCompat = NotificationManagerCompat.from(this)
notificationsManagerCompat.areNotificationsEnabled() // 실제로 알림이 활성화 되어 있는지를 확인하는 AndroidX 함수 
```
미디어 세션과 관련된 알림은 이 동작 변경사항과 무관  
포그라운드 서비스 알림은 대체로 영항을 받는다. 

### Revocking permissions/clearing flags using ADB
```
adb shell pm revoke PACKAGE_NAME 
android.permission.POST_NOTIFICATIONS

adb shell pm clear-permission-flags PACKAGE_NAME \ 
android.permission.POST_NOTIFICATIONS user-set

adb shell pm clear-permission-flags PACKAGE_NAME \
android.permissions.POST_NOTIFICATIONS user-fixed 
```
------
## Explicit Intents (all)
- Intent Filters   
    Must match intent-filter element to be delivered to any app targeting SDK 33   
    명시적 인텐트를 SDK 33 targeting 하는 다른 앱으로 보내면, 인텐트를 받는 앱에서 인텐트 필터 요소와 일치하는 경우에만 해당 인텐트 전달  

### Manually filtering non-exported intents
exported intent와 non-exported intent 에 한가지 핸들러를 사용할 경우 허용되지 않는 액션을 필터링하는 약간의 코드를 수신기에 추가하면 외부 앱에서 내부 코드를 갑자기 트리거하는 문제를 방지할 수 있습니다. 

```kotlin
class ExternalReceiver : BroadcastReceiver() {
    private val allowedActions = listOf(
        "com.example.PUBLIC_EXT_ACTION_1", "com.example.PUBLIC_EXT_ACTION_2")
    )

    override fun onReceive(context: Context, intent: Intent) {
        val action = intent.action ?: return //skip when action is null

        //only accept actions tat are allowed
        if (!allowedActions.contains(action)) { //filtering happening here!
            return 
        }

        //Displatch all actions to centeralized handler
        myIntentHAndler.handleIntent(context, intent)
    }
}
```

------
## Protect private clipboard data (all)
비밀번호 팝업에서 감추는 기능 추가 
```kotlin
//when your app targets SDK version 33 or higher
clipData.apply {
    discription.extras = PersistableBundle().apply {
        putBoolean(ClipDescription.EXTRA_IS_SENSITIVE, true)
    }
}

// If you app targets a lower SDK version 
clipData.apply {
    discription.extras = PersistableBundle().apply {
        putBoolean("android.content.extra.IS_SENSITIVE", true)
    }
}
```
![image](https://user-images.githubusercontent.com/360685/208288610-fc738d0a-7d84-4426-a76f-9bccbf5fd562.png)

------
## Migrate from deprecated sharedUserId (all)
Manifest 에서 공유된 사용자 ID 를 옮기면 앱 업데이트가 차단되는 문제가 발생해서 마이그레이션 하는 수단 추가 
```xml
<!-- To make sure your app still receives updates, continue to use "android:shareUserId" if already in the manifest -->
<manifest
    android:sharedUSerId="SHARED_PACKAGE_NAME"
    android:sharedUserMaxSdkVersion="32"> // only used when installing on SDK versions < 33
```

------
Target 33 은 2023년 11월 까지 

![Migrate your apps to Android 13 8-45 screenshot](https://user-images.githubusercontent.com/360685/208289000-4ce073b7-b88f-48e5-9201-1c84b52bcdbe.png)

# Taget SDK 33 변경사항 
## Nearby WiFi Devices
- 이전버전 : 일반적인 Wifi 사용 사례는 사용자가 앱에 액세스 권한을 부여해야 위치를 찾을 수 있었음.
- Android 13:  주변 기기 그룹에 주변 Wifi 기기 런타임 권한 도입   
     스캔 시작, 스캔 결과와 같이 Wifi 관리자 클래스의 메서드를 앱에서 사용하지 않는다는 정보를 매니페스트에 포함하면 주변 Wifi 기기 권한만 사용 가능 
```xml

<uses-permission
    android:name="android.permission.NEARBY_WIFI_DEVICES"
    android:usesPermissionFlags="neverForLocation">

<uses-permission
    android:name="android.permission.ACCESS_COARSE_LOCATION"
    android:maxSdkVersion="32"/>
<uses-permission
    android:name="android.permission.ACCESS_FINE_LOCATION"
    android:maxSdkVersion="32"/>

```
![Migrate your apps to Android 13 9-30 screenshot](https://user-images.githubusercontent.com/360685/208289261-2fbe5b0c-260c-41ff-82b8-f6a30889276c.png)

-----
## Granular Media Permissions
No more READ_EXTERNAL_STORAGE
외부 스토리지 읽기 권한 대신 하나 이상의 세분화된 미디어 권한을 요청
![Migrate your apps to Android 13 9-42 screenshot](https://user-images.githubusercontent.com/360685/208289326-2b4ff772-98c9-4079-af2c-d4653d7a96fd.png)

------
## Background sensing
```xml
<uses-permission
    android:name="android.permission.BODY_SENSORS">
<uses-permission
    android:name="android.permission.BODY_SENSORS_BACKGROUND"> // 백그라운드 권한 추가 
```
패키지 설치 프로그램 허용 목록이 있는 제한된 권한 : Health Connect 사용 

------
### 대체된 API 
![Migrate your apps to Android 13 10-31 screenshot](https://user-images.githubusercontent.com/360685/208289454-ea164bee-bcc0-43fa-a9fb-2fc117617f63.png)

------
## Deferred Boot Complted
Target 33 이면 앱이 제한된 상태일 때 앱이 다른 이유로 다시 시작될 때까지 BOOT_COMPLETED, LOCKED_BOOT_COMPLETED 를 전송하지 않음   

When in restricted app bucket
- BOOT_COMPLETED
- LOCKED_BOOT_COMPLETED

Testing with compatibility framework  
설정 또는 ADB 를 통해서 앱의 대상 SDK 버전을 바꾸지 않고도 특정 동작 변경 사항을 전환할 수 있다. 
- Test behavior changes without changing targetSdkVersion DEFER_BOOT_COMPLETED_BROADCAST_CHANGE_ID : 사용자가 앱 위젯을 추가하면 앱은 절대 제한된 상태로 들어가지 않는다. 

------
## 사용자 경험 개선 사항 

### 의미 순서대로 미디어 컨트롤 표시 
이전에는 미디어 스타일에서 추가된 순서대로 최대 5개 컨트롤 표시
![Migrate your apps to Android 13 11-38 screenshot](https://user-images.githubusercontent.com/360685/208289713-694f4915-4f60-4b4e-82b4-37eff23590b3.png)

### WebView 에서 setForceDark 메서드 무시   
앱 테마 속성에 따라 미디어 쿼리의 색 구성표 설정 
![Migrate your apps to Android 13 12-3 screenshot](https://user-images.githubusercontent.com/360685/208289763-54b280d3-1c1f-43d9-932f-142e92f2689a.png)

### Advertising ID permission
광고 ID 사용에 대한 권한 추가   
sdk 33 을 타게팅하면 라이브러리 매니페스트에서 이 권한이 자동 요청 
```xml 
<uses-permission
    android:name="com.google.android.gms.permission.AD_ID">
```
-----
# What's Next? 
## Photo Picker
스토리지 권한 제거 후 Photo picker 로 변환 
![Migrate your apps to Android 13 13-14 screenshot](https://user-images.githubusercontent.com/360685/208289915-a95c1049-96ef-4ee4-a998-9519756b18e2.png)

사진 선택 도구를 사용할 수 없을 경우 action_open_document 를 사용하도록 변경 
![Migrate your apps to Android 13 13-40 screenshot](https://user-images.githubusercontent.com/360685/208289982-43435663-1788-43fa-bcf8-e784bd5f01fe.png)

![Migrate your apps to Android 13 13-53 screenshot](https://user-images.githubusercontent.com/360685/208290054-fe67b96d-2e28-4f4e-b722-1ac2e303999a.png)

`isPhotoPickerAvailable()`은 내부적으로 `getExtensionVersion()` 을 호출하는데, 이방법을 통해 런타임에서 SDK 확장 프로그램 존재를 확인 가능 
![Migrate your apps to Android 13 14-14 screenshot](https://user-images.githubusercontent.com/360685/208290058-74b00c89-e9e9-4a40-ac5f-2914474e5cc2.png)

외부 스토리지 읽기와 같은 런타임 권한을 android 13에서 더이상 사용하지 않는다면 앱이 스스로 권한을 취소할 수 있다. 

![Migrate your apps to Android 13 14-24 screenshot](https://user-images.githubusercontent.com/360685/208290060-2ccd4f5b-90f0-4843-877b-4a3a10a6af9d.png)

----
## Others
![Migrate your apps to Android 13 15-21 screenshot](https://user-images.githubusercontent.com/360685/208290266-74df5d1c-483e-446d-a6a0-d4ab531e0685.png)

![Migrate your apps to Android 13 16-0 screenshot](https://user-images.githubusercontent.com/360685/208290311-f82ec535-8f50-440d-b8c0-f3d18a407cf8.png)

### Setting per-app language preferences
Android X 의 App Compat API 를 사용하면 Android 13을 쉽게 지원하면서도 Android 4.2 까지 호환성 제공 
```kotlin
val appLocale = LocaleListCompat.forLanguageTags(it.localeCode)
AppCompatDelegate.setApplicationLocals(appLocale)

// add androidx.appcompat.app.AppLocalesMetadataHolderService declaration to Android manifest to auto-persist setting on SDK version < 33
```

![Migrate your apps to Android 13 16-53 screenshot](https://user-images.githubusercontent.com/360685/208290767-74a5f254-bc7b-41a5-ae57-8ebfebf5998c.png)

![Migrate your apps to Android 13 17-4 screenshot](https://user-images.githubusercontent.com/360685/208290764-9a9ddb50-c523-44ed-9cd7-be48d33b8c41.png)
----

## Conclusion
### Test 
Take your migraion one step at a time and make sure to test
### Optimize
Minimize your app impact on system health + take advantage of new capabilites
