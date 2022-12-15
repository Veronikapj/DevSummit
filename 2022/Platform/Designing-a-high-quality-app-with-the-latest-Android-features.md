## Designing a high quality app with the latest Android features

.

![image-20221215173940300](/Users/kwonhyeokjun/Library/Application Support/typora-user-images/image-20221215173940300.png)



### Edge to Edge

특별한 레이아웃 혹은 지도와 같이 확장된 UI가 필요한 경우,

넓은 핸드폰 화면(폴더블, 테블릿 )

"네비게이션 바를 투명하게 해주세요!" 라는 요청이 있었습니다.

WindowInsets(platform API in Android 11+) / WindowInsetsCompat(AndroidX 1.5.0-alpha01 or higher) / WindowCompat(AndroidX 1.5.0-alpha01 or higher) 를 사용하여 화면을 개발자의 의도대로 구성해줄 수 있습니다. [Android 10,11]

eg. 화면이 내려가면 AppBar 사라짐 / status와 nav bar 색을 동일하게 구성하며 일치된 느낌을 줌

![image-20221215182142404](/Users/kwonhyeokjun/Library/Application Support/typora-user-images/image-20221215182142404.png)


![image-20221215182641619](/Users/kwonhyeokjun/Library/Application Support/typora-user-images/image-20221215182641619.png)

![image-20221215182744791](/Users/kwonhyeokjun/Library/Application Support/typora-user-images/image-20221215182744791.png)

모바일에서만 사용되는 것이 아니라 폴더블, 태블릿 등 다양하게 적용이 가능하다



### Predictive back

android 13으로 넘어오면서 back guesture를 하는 경우, 현재 화면이 작아지는 효과적인 UI를 볼 수 있다.

![image-20221215182949804](/Users/kwonhyeokjun/Library/Application Support/typora-user-images/image-20221215182949804.png)


Android13

<application
    android: enableOnBackInvokedCallback = "true"> 추가해주기

![image-20221215183309579](/Users/kwonhyeokjun/Library/Application Support/typora-user-images/image-20221215183309579.png)



### Accessible color system

Dynamic Color : Material Color에 대해 커스터마이즈가 가능. API for Android 12+

- Aceesble : Disgn for everyone
- Scalable : Design tokens enable flexiblility and consistency(color 시스템(color token)을 통일하면서 디자이너와 협업이 쉬워짐)

작업 중 입니다..