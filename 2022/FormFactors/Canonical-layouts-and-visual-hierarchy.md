# Canonical layouts and visual hierarchy
Designing for larger screens 

https://youtu.be/FrkIa9vZjCI

## Canonical Layouts
- List-detail
- Supporting Panel
- Feed
![Canonical layouts and visual hierarchy_ Designing for larger screens 1-59 screenshot](https://user-images.githubusercontent.com/360685/206890213-3864990a-1691-4fcc-83e2-ef659ee20f3c.png)

### Layout Regions
![Canonical layouts and visual hierarchy_ Designing for larger screens 2-12 screenshot](https://user-images.githubusercontent.com/360685/206890285-e088e219-6a73-464e-8ac4-16dca28e461a.png)
1. the Navigation region
    - Navigation rail or Navigation drawer
2. the App bar region
    - 구성 요소 및 작업을 표시하고 그룹화 
    - 주요 작업을 수행하는데 도움이 되는 내부 요소 등 
3. Body 
    - 앱 대부분의 콘텐츠 

![Canonical layouts and visual hierarchy_ Designing for larger screens 3-43 screenshot](https://user-images.githubusercontent.com/360685/206890443-27645d15-63af-495c-930b-ab1435f87c20.png)

## 1. List-Detail
![Canonical layouts and visual hierarchy_ Designing for larger screens 4-38 screenshot](https://user-images.githubusercontent.com/360685/206890498-d1180cda-cea6-4b99-8c98-8be8382898f3.png)

### Different Activity - Activity Embedding
Android 12L and later 
![Canonical layouts and visual hierarchy_ Designing for larger screens 6-51 screenshot](https://user-images.githubusercontent.com/360685/206890679-9364aacd-d50b-4d73-9ed4-10c7a3ea7c4f.png)

```xml
<!-- Automatically split the following Activity pairs-->
<SplitPairRule
    window:splitRatio="0.3"
    window:splitMinWidth="600dp">
    <SplitPairFilter
        window:primaryActivityName=".MainActivity"
        window:secondaryActivityName=".DetailActivity" />
</SplitPairRule>

<SplitPlaceholderRule
    window:placeholderActivityName=".PlaceholderActivity">
</SplitPlaceholderRule>    
```

### SlidingPaineLayout - View based solution
![Canonical layouts and visual hierarchy_ Designing for larger screens 8-46 screenshot](https://user-images.githubusercontent.com/360685/206891001-44384476-ec47-4566-a6ec-819e21bd117f.png)

## 2. Supporting Panel
![Canonical layouts and visual hierarchy_ Designing for larger screens 9-42 screenshot](https://user-images.githubusercontent.com/360685/206891101-fa9771ae-0171-48d4-a6a7-d8fb9a0fe3ea.png)

![Canonical layouts and visual hierarchy_ Designing for larger screens 10-10 screenshot](https://user-images.githubusercontent.com/360685/206891184-a68faa1c-d381-4ecc-89c6-60906c513c46.png)

If you have a **primary-secondary-conent** relationship within your UI in layouts, supporting panel is a great largescreen optimized layout choice for you 


### Compose + TwoPaine
using Accompanist
```kotlin
//SupportingPanel.kt

@Composable
fun SupportingPanel(
    main: @Composable () -> Unit,
    supporting: @Composable () -> Unit, 
    windowSizeClass: WindowSizeClass,
    displayFeatures: List<DisplayFeature>
) {
    TwoPain(
        first = main,
        second = supporting, 
        strategy = when(windowSizeClass.widthSizeClass) {
            WindowWidthSizeClass.Compact -> VerticalTwoPainStrategy(0.5F)
            WindowWidthSizeClass.Medium -> HorizontalTwoPainStrategy(0.5F)
            else -> HoritonzontalTwoPainStrategy(0.7F)
        },
        displayFeatures = displayFeatures
    )
}

```

### Resource Qualifiers - View based solution 
![Canonical layouts and visual hierarchy_ Designing for larger screens 12-7 screenshot](https://user-images.githubusercontent.com/360685/206891560-7fc37168-4709-443f-a751-1b2df668f8e1.png)

![Canonical layouts and visual hierarchy_ Designing for larger screens 12-29 screenshot](https://user-images.githubusercontent.com/360685/206892559-3eb32fcb-29e0-4c44-a8c1-f8be1f2b8103.png)

## isTablet?
**Don't**   
너비로만 판단하게 되면 데스크탑과 같은 상황 - 사용자가 앱 width와 height를 조정할 수 있음 - 에서 동작이 이상해질 수 있다. 
```XML
// sw720dp/bools.xml
<resource>
    <bool name="isTablet">true</bool>
</resource>
```
더 넓은 화면 840dp
![Canonical layouts and visual hierarchy_ Designing for larger screens 13-34 screenshot](https://user-images.githubusercontent.com/360685/206892809-80d096e7-665b-466e-86c4-0fdecc4417c9.png)

Resource Qualifiers 를 사용하는 경우, 동일한 레이아웃 요소를 여러벌 관리하게 되므로 업데이트를 다 같이 해주어야 한다. 

## Feed 
If you have content of **equal weight or eqaul relationships**, or maybe you want to highlight one or the other, but generally, they are of equal importance. 
![Canonical layouts and visual hierarchy_ Designing for larger screens 14-30 screenshot](https://user-images.githubusercontent.com/360685/206893043-0be9ce1c-d037-488a-ae9c-eaf9f64250e2.png)

### Compose
```kotlin
@Composable
fun remeberColumns(windowSizeClass: WindowSizeClass) = remeber(windowSizeClass) {
    when(windowSizeClass.widthSizeClass) {
        WindowSizeClass.Compact -> GridCells.Fixed(1)
        WindowSizeClass.Medium -> GridCells.Fixed(2)
        else -> GridCells.Adaptive(240.dp)
    }
}

```

![Canonical layouts and visual hierarchy_ Designing for larger screens 16-50 screenshot](https://user-images.githubusercontent.com/360685/206893395-524d343b-7a17-4913-9ce5-186969130818.png)


### RecyclerView 
```kotlin
override fun onCreateView(...): View? {
    val columnCount = resource.getInteger(R.integer.column_count)
    with(view) {
        layoutManager = when {
            columnCount <= 1 -> LinearLayoutManager(context)
            else -> GridLayoutManager(context, columnCount)
        }
    }
}
```
```xml
//integers.xml
<integer name ="column_count">1</integer>

//w600dp/integers.xml
<integer name ="column_count">2</integer>

//w840dp/integers.xml
<integer name ="column_count">5</integer>
```