# Everything about storage on Android

https://youtu.be/jcO6p5TlcGs

## Storage Architecture
Where files are stored on Android 

플래시 메모리에 저장된 기본 스토리지 볼륨 또는 OS 와 App 이 있는 하드 드라이브에 위치 
![Everything about storage on Android 0-33 screenshot](https://user-images.githubusercontent.com/360685/208291759-04a2c53c-68ef-470e-b071-d40380c73697.png)

앱을 설치하면, 같은 기본 스토리지에 앱의 internal directory 를 생성  
앱만 접근 가능. confidential, databases, configuration files 은 이곳에 저장.
![Everything about storage on Android 0-42 screenshot](https://user-images.githubusercontent.com/360685/208291831-1a0dadac-ad24-47cf-a9a4-28b26b5fb60e.png)

external storage 생성   
일부 디바이스에서는 기본 스토리지에 생성 제한되는 경우가 있어서 micro SD card 를 통해서만 앱 외부 저장소를 사용할 수도 있다.   
지워질 수도 있어서 접근하기 전에 항상 접근이 가능한지 체크해야 한다. 안드로이드 11 이전에는 다른 앱에서 읽거나 쓰는 접근이 가능했기 때문에, 이곳에 민감한 정보를 저장해서는 안된다. 
![Everything about storage on Android 1-28 screenshot](https://user-images.githubusercontent.com/360685/208292335-489a18b5-409f-45e9-8248-d8468851bcb3.png)

internal & external storage 는 앱 삭제 시에 지워진다. 
![Everything about storage on Android 1-42 screenshot](https://user-images.githubusercontent.com/360685/208292513-977d9dfe-4452-4faa-8d03-74919db1461e.png)

Shared Storage   
파일관리자, 또는 포토갤러리 앱 같은 모든 앱의 공통 위치. 앱을 지워도 삭제되지 않는다. 
![Everything about storage on Android 1-50 screenshot](https://user-images.githubusercontent.com/360685/208292549-dc95eea8-240d-419a-981d-c2ec10355b92.png)

Android 버전에 따라 파일 경로가 달라질 수 있기 떄문에 절대경로를 저장하면 안되고, 상대 경로를 사용해야 한다. 
![Everything about storage on Android 2-21 screenshot](https://user-images.githubusercontent.com/360685/208293023-96e8d8ba-0e15-4a54-832b-ce16e2f7074d.png)

## Scoped Storage
Focus on privacy & user control

### Up to Android 9
Shared storage was readable and writable by apps that requested the right permissions.
### Android 10+
- Scoped access  
    Limited access only media files like photos, videos and autio files when requesting READ_EXTERNAL_STORAGE.   
- SAF for document files
    Document files are accessible through the Storage Access Framework via the document picker.   
    기타 documents 는 프레임워크의 document picker 를 통해 접근

### Android 10+ 변경사항 
- Permission reduction   
    WRITE_EXTERNAL_STORAGE permission isn't needed to add files to the shared storage anymore 
- Explicit consent    
    Dialog shown to users when app wants to modify or delete a media file
- Privacy friendly  
    Location metadata are automatically stripped from media files unless the ACCESS_MEDIA_LOCATION is required

-----
## Use Cases
Common stoarge usage and which API to use

### Downloading a file to internal storage 

```kotlin
val client = OkHttpClient()
val request = Request.Builder().url(CONFIG_URL).build()

client.newCall(request).execute().use { response -> 
    // use : 메모리 누수를 방지하기 위해 람다 끝에서 기본 네트워크 소켓을 자동으로 닫는다. 
    response.body?.byteStream()?.use { input ->
        val target = File(context.filesDir, "user-config.json")
        target.outputStream().use { output ->
            input.copyTo(output)
        }
    }
}
```

### Choosing destination based on available space
```kotlin
val fileSize = 500_000_000L // 500MB

val target = if (context.fileDir.usableSpace > fileSize) {
    context.filesDir
} else {
    context.getExternalFilesDirs(null).find { externalStorage ->
        externalStorage.usableSpace > fileSize
    } 
} ?: throw IOException("Not enough space")

val file = File(target, "big-file asset")
```

### Add image to shared storage 
```kotlin
// Request WRITE_EXTERNAL_STARGE up to Android 9

val type = Environment.DIRECTORY_PICTURES //video, document 도 가능 
val target = Environment.getExternalStoragePublicDirectory(type)
val newImage = File(target, "new-image.jpg")

// Copy content into newFile using streams
MediaScannerConnection.scanFile(
    context,
    arrayOf(newImage.path),
    arrayOf("image/jpeg")
) { path, uri -> 
    Log.d("MediaScanner", "newImage: $path || $uri")
}

```
Android 10 이상에서 즐겨찾기, 휴지통, 대기 중인 파일과 같은 더 많은 기능에 액세스하고 최상의 성능을 얻으려면 파일 API에 의존하는 대신 MediaStore API를 사용하여 공유 저장소에 파일을 추가하는 것이 좋습니다.

### Select a file with the document picker 
```kotlin
// Add the Jetpack Activity dependency 
val document picker = rememberLauncherForActivityResult(OpenDocument()) { uri ->
    if (uri == null) return 
    
    context.contentResolver.openInputStream(uri)?.use {
        //we can new copy the file content
    }
}

//usage
documentPicker.launch(arrof("application/pdf"))
```

안드로이드 12L 이상에서 Shared Storage 에 있는 포토와 비디오에 접근하려면 read external storage 권한을 요청하고 아이템 선택을 위한 UI를 제공해야 합니다. 좀 더 쉽고 간단하게 제공할 수 있는 방법을 소개합니다. 

### Photo Picker
![Everything about storage on Android 9-35 screenshot](https://user-images.githubusercontent.com/360685/208295252-7c27ded1-4dd1-4727-84e2-d3548529b2cb.png)

```kotlin
// Add the Jetpack Activity 1.6.0+ dependency

//Register the photo picker ActivityResult
val picker = rememberLauncherForActivityResult(
    PickVisualMedia()
) { uri ->
    Log.d("Media selected: $uri")
}

//open the photo picker
picker.launch(PickVisualMediaRequest(PickVisualMedia.ImageAndVideo))
```

If you need access to a Uri even after your app is closed
```kotlin
contentResolver.takePersistableUriPermission(
    photoUri,
    FLAG_GRANT_READ_URI_PERMISSION
)
```

## Others
- Privacy & Transparency : Privacy & transparency have been tremendously improved over the recent releases of Android, with even UX enhancements like the photo picker
- Permissions-less APIS : We're working on adding more APIs that keep the user in control of giving access while removing the need to request permissions on apps side
- Uri preferred approach : Rather than relying on file paths, which only offer access to local files and can be canged, we're recommending to use Uri to access files on Android 
- goo.gle/android-data-storage
