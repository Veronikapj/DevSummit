# Styling text in Compose
https://youtu.be/_qls2CEAbxI

## Typography 구현

```kotlin
val jetchatTypography = Typography (
    bodyLarge = TextStyle(
        fontFamily = fontFamily,
        fontWeight = FontWeight.Mormal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.15.sp
    ),
    bodyMedium = ...,
    bodySmall = ...,
    headlineLarge = ...,
    headlineMedium = ...
)
```
Text Composable 에 적용 
![Styling text in Compose 1-49 screenshot](https://user-images.githubusercontent.com/360685/205482190-fa56e261-9f8b-4dbf-9907-912d1ee3e4c0.png)

## Button

![Styling text in Compose 2-8 screenshot](https://user-images.githubusercontent.com/360685/205482281-b578584b-240f-43eb-bba2-99a2f8f47301.png)
![Styling text in Compose 2-28 screenshot](https://user-images.githubusercontent.com/360685/205482283-19295f96-3e79-48ff-b78b-08c8de417340.png)

`LocalTextStyle.current`를 lableLarge로 구성하는 컴포지션 로컬을 생성하는 `ProvideTextStyle` 관련 코드를 찾을 수 있다. TextComposable 을 보면 스타일이 `LocalTextStyle.current` 로 설정되어 있어서 버튼의 설정 값을 따라간다. 

```kotlin
Text.kt

@composable
fun Text(
    ...
    style: TextStyle = LocalTextStyle.current
) { 
    ... 
}
```

## 텍스트 더보기 
![Styling text in Compose 3-9 screenshot](https://user-images.githubusercontent.com/360685/205482535-df3b6947-1373-449f-a7d6-0ffc68236a96.png)
![Styling text in Compose 3-13 screenshot](https://user-images.githubusercontent.com/360685/205482534-6d8baaf3-5794-4146-a189-8c7b48fa88ff.png)

```kotlin
var showMore by remember { mutableStateOf(false) }

Text(
    text = styledMessage,
    maxLines = 8, 
    overflow = TextOverflow.Ellipsis,
    onTextLayout = {
        if (it.hasVisualOverflow) {
            showMore = true
        }
    }
)

if (showMore) {
    Button(...)
}

```

## 글꼴 변경 
![Styling text in Compose 4-42 screenshot](https://user-images.githubusercontent.com/360685/205482714-d09d811d-a958-4e10-8261-e7bab32d74aa.png)

글꼴이 로드 되는지 미리 로드 해서 사용하기 
```kotlin
in composition

val fontFamilyResolver = LocalFontFamilyResolver.current
val fontFamily = FontFamily(...)

val coroutineScope = remeberCoroutineScope()

LaunchedEffect(Unit) {
    coroutineScope.launch(Dispatchers.IO) {
        fontfamilyResolver.preload(fontfamily = fontFamily)
    }
}
```

### 가변 글꼴 
파일을 로컬로 다운로드하고 FontVariation API 사용하여 필요한 값 설정한 후 글꼴 모음에 추가한다. 
![Styling text in Compose 5-51 screenshot](https://user-images.githubusercontent.com/360685/205482988-90de0bb9-fc59-4593-8874-c5f2e9be739f.png)

## AnnotatedString
View System 의 SpannableString 과 같은 동작 
![Styling text in Compose 6-33 screenshot](https://user-images.githubusercontent.com/360685/205483073-33ef10f5-a94b-4332-8015-41e855fce75d.png)

## TextField (= EditText) 사용하기 
![Styling text in Compose 7-1 screenshot](https://user-images.githubusercontent.com/360685/205483147-3971b72b-a1b4-43fb-9b25-38931cc45e4b.png)
![Styling text in Compose 7-42 screenshot](https://user-images.githubusercontent.com/360685/205483146-a7a6c2a5-74d5-4ea5-af0b-6d1426280989.png)


## Material 디자인이 아니거나 더 커스텀 하기를 원하는 경우 
![Styling text in Compose 8-10 screenshot](https://user-images.githubusercontent.com/360685/205483277-81f6eb9b-a9ff-45c4-b630-582501908a9c.png)

BasicTextField  
줄 끝에 가까워질 수록 커서 색상이 바뀌는 디자인
![Styling text in Compose 8-32 screenshot](https://user-images.githubusercontent.com/360685/205483145-f057fc66-988f-4d17-8664-2f365c3d7372.png)

완전한 사용자 정의 
![Styling text in Compose 9-8 screenshot](https://user-images.githubusercontent.com/360685/205483143-66fdd12c-845b-4480-bb32-02bd90ad87f8.png)
