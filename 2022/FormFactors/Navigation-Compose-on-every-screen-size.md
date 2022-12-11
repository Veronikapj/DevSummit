# Navigation Compose on every screen size


- navController : navigation의 state 관리 
  - this is how you navigate between different destinations. and also maintatins the backstack.
- navigation Graph : navController위한 map 제공   
  - This is where you define all of your destinations. and how they relate to one another
- navHost : 지금 뭐할지에 초점이 맞춰져있음. 
  - the NavHost is a bounding box container for the part of your UI that should be considered part of navigation

```kotlin
@Composable
fun NavGraph() {
    val navController = rememberNavController()
    NavHost(navController = navController, startDestination = "screen_one") {
        composable("screen_one") { ScreenOne(navController) }
        composable("screen_two") { ScreenTwo(navController) }
    }
}
```

<img width="846" alt="스크린샷 2022-12-11 오후 5 42 38" src="https://user-images.githubusercontent.com/70066242/206902290-87a68833-0a6b-4a8c-a057-ccdf390d1023.png">

- BottomNavigation : 하단에 존재.
- NavigationRail : side에 존재, medium 혹은 expanded size에서 확인 가능 

