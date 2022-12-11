# Build better UIs across form factors with Android Studio 
https://youtu.be/Z4h9cvlE66E

## Window Size Classes 
An easy categorization to create adaptive layouts that look greak on all screen sizes.
- Compact < 600dp
- Medium 600~860dp
- Expanded > 840dp 

![Build better UIs across form factors with Android Studio 1-3 screenshot](https://user-images.githubusercontent.com/360685/206887904-856e8004-63d9-4be8-a971-23cc1b630a5a.png)

## View System 

### 1. Reference devices
- Phone 
- Foldable / small tablet 
- Tablet
- Desktop

![Build better UIs across form factors with Android Studio 2-11 screenshot](https://user-images.githubusercontent.com/360685/206887948-95a7bbed-d097-4a09-8abf-d9b6ea2fea31.png)

### 2. Visual Linting
- 여러가지 폼 펙터 레이아웃에서 발생할 수 있는 디자인 이슈들을 찾습니다. 

![Build better UIs across form factors with Android Studio 4-25 screenshot](https://user-images.githubusercontent.com/360685/206888011-7e695866-48e0-484e-8299-4cd30c6a1c21.png)

![Build better UIs across form factors with Android Studio 4-36 screenshot](https://user-images.githubusercontent.com/360685/206888016-12d6b0ac-6d2a-4db4-939c-cb512b535b7c.png)


#### Fix the Bottom Navigation issue
![Build better UIs across form factors with Android Studio 5-18 screenshot](https://user-images.githubusercontent.com/360685/206888041-b089d88e-f7c0-4fa2-a8aa-1b172e2771f6.png)

### 3. Layout Validation
![Build better UIs across form factors with Android Studio 5-59 screenshot](https://user-images.githubusercontent.com/360685/206888083-786a2f73-6044-478a-90f3-32aa418705f1.png)

## Compose
```kotlin

implementation "androidx.compose.material3:material3-window-size-class:<...>"

@Composable
fun Main(windowSizeClass: WindowSizeClass) {
    if (windowSizeClass.widthSizeClass == WindowWidthSizeClass.Compact) {
        // use a bottom navigation bar
    } else {
        // use a navigation rail
    }
}

//from MainActivity
class MainActivity : ComponentActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MainScreen(calculateWindowSizeClass(activity = this))
        }
    }
}

```

### Compose Preview
![Build better UIs across form factors with Android Studio 8-30 screenshot](https://user-images.githubusercontent.com/360685/206888575-f06c677d-7e9c-4646-a837-1b557261843b.png)


### Compose Multi-preview
A convenient way to define preview groups that can easily be reused, perfect for testing adaptive layouts
```kotlin
    @PreView(name = "phone", device = Devices.PHONE)
    @PreView(name = "foldable", device = Devices.FOLDABLE)
    @PreView(name = "custom", device = "spec:width=1280dp,height=800dp,dip=480")
    @PreView(name = "desktop", device "id:desktop_medium")
    annotation class RefrenceDevices

    @ReferenceDevices
    @Composable
    fun MainPreview() {
        val config = LocalConfiguration.current
        val windowSizeClass = WindowSizeClass.calculateFromSize(
            DpSize(config.screenWidthDp.dp, config.screenHeightDp.dp)
        )
        Main(windowSizeClass)
    }
```

## Resizeabke Emulator
- phone, foldable, tablet 모드를 빠르게 변경 가능
![image](https://user-images.githubusercontent.com/360685/206888801-4362f45b-1b4e-4da5-ae38-5057204edfc3.png)

## Others
- Desktop AVDs : 데스크탑 에뮬레이터 추가 
- Visual Linting for Wear OS : Wear OS 를 위한 디자인 lint 추가 
- Wear OS Preview
    ![Build better UIs across form factors with Android Studio 15-36 screenshot](https://user-images.githubusercontent.com/360685/206888892-47eb9efa-dcac-4fa0-8980-4f8661f56434.png)

## Recap
![Build better UIs across form factors with Android Studio 19-2 screenshot](https://user-images.githubusercontent.com/360685/206888903-7115878e-fe3f-4af1-a9b5-4cf29e15167d.png)
