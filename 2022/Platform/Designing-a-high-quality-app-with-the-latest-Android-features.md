## Designing a high quality app with the latest Android features

.
<img width="729" alt="스크린샷 2022-12-15 오후 5 39 36" src="https://user-images.githubusercontent.com/70066242/207836748-ab58dfe1-9d77-45e8-abb4-88782bb9e834.png">



### Edge to Edge

특별한 레이아웃 혹은 지도와 같이 확장된 UI가 필요한 경우,

넓은 핸드폰 화면(폴더블, 테블릿 )

"네비게이션 바를 투명하게 해주세요!" 라는 요청이 있었습니다.

WindowInsets(platform API in Android 11+) / WindowInsetsCompat(AndroidX 1.5.0-alpha01 or higher) / WindowCompat(AndroidX 1.5.0-alpha01 or higher) 를 사용하여 화면을 개발자의 의도대로 구성해줄 수 있습니다. [Android 10,11]

eg. 화면이 내려가면 AppBar 사라짐 / status와 nav bar 색을 동일하게 구성하며 일치된 느낌을 줌


<img width="838" alt="스크린샷 2022-12-15 오후 6 21 37" src="https://user-images.githubusercontent.com/70066242/207836889-31abd0fa-392a-4dbc-9edf-1b9a81050881.png">
<img width="809" alt="스크린샷 2022-12-15 오후 6 26 32" src="https://user-images.githubusercontent.com/70066242/207837001-b012eb13-13ed-4d6e-9081-0d2c3e718255.png">

<img width="372" alt="스크린샷 2022-12-15 오후 6 27 37" src="https://user-images.githubusercontent.com/70066242/207837020-31f9c60d-adc2-427b-96c0-32c4f952379e.png">


모바일에서만 사용되는 것이 아니라 폴더블, 태블릿 등 다양하게 적용이 가능하다


### Predictive back

android 13으로 넘어오면서 back guesture를 하는 경우, 현재 화면이 작아지는 효과적인 UI를 볼 수 있다.

<img width="766" alt="스크린샷 2022-12-15 오후 6 29 40" src="https://user-images.githubusercontent.com/70066242/207837063-55520fd1-6d7a-4eac-8a35-c919bfb4ef5d.png">


Android13

<application
    android: enableOnBackInvokedCallback = "true"> 추가해주기


<img width="855" alt="스크린샷 2022-12-15 오후 6 33 03" src="https://user-images.githubusercontent.com/70066242/207837099-c9510e16-72e7-4ee5-baba-a2c0177998b6.png">


### Accessible color system

Dynamic Color : Material Color에 대해 커스터마이즈가 가능. API for Android 12+

- Aceesble : Disgn for everyone
- Scalable : Design tokens enable flexiblility and consistency(color 시스템(color token)을 통일하면서 디자이너와 협업이 쉬워짐)

작업 중 입니다..
