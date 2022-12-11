# Navigation Compose on every screen size


- navController : navigation의 state 관리 / this is how you navigate between different destinations. and also maintatins the backstack.
- navigation Graph : navController위한 map 제공 / This is where you define all of your destinations. and how they relate to one another
- navHost : 지금 뭐할지에 초점이 맞춰져있음. the NavHost is a bounding box container for the part of your UI that should be considered part of navigation



Outside the NavHost

- TopAppBar
- BottomNavigation : 하단에 존재.
- NavigationRail : side에 존재 -> medium 혹은 expanded size에서
- DrawerLayout



BottomNavigation 에서 NavigationRail 으로 전환 

