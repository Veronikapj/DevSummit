# More performance tips for Jetpack Compose

https://youtu.be/ahXLwg2JYpc

## Defer reading state
![More performance tips for Jetpack Compose 4-19 screenshot](https://user-images.githubusercontent.com/360685/205478065-b2469f54-2f6f-4ef8-9e4f-2c79e386ceda.png)
컴포지션은 컴포저블 트리를 구축하여 표시할 항목을 결정합니다. 

![More performance tips for Jetpack Compose 4-21 screenshot](https://user-images.githubusercontent.com/360685/205478084-d1775b7f-915f-4a1a-915a-93adec7a811c.png)
![More performance tips for Jetpack Compose 4-27 screenshot](https://user-images.githubusercontent.com/360685/205478089-0f60369b-d873-4beb-9793-e6b0f0773d63.png)

```kotlin
@Composable
fun Parent() {
    val offset by animateFloatAsState(targetValue = 10f)
    Column {
        Header()
        Child(offset) // 값을 부모에서 읽고 있음 -> Parent recomposition
        Footer()
    }
}

@Composable
fun Child(offset: Float) {
    Box(Modifer.offset(y = offset.dp))
}
```

offset state 값 읽는 겻을 연기할 수 있다. 가능한 상태를 늦게 읽자.

X 나쁜 예시? 실제 State 개체를 전달?
```kotlin
@Composable
fun Parent() {
    val offset = animateFloatAsState(targetValue = 10f)
    Column {
        Header()
        Child(offset) 
        Footer()
    }
}

@Composable
fun Child(offset: State<Float>) {
    Box(Modifer.offset(y = offset.value))
}
```
state 읽는 것을 지연시키긴 하지만 코드 유지보수가 어려워진다. 
by delegate 를 사용할 수 없고 `.value` 를 추가해야 한다. 또한 고정 값을 전달할 수 없다. 

### 람다를 사용하자!
```kotlin
@Composable
fun Parent() {
    val offset by animateFloatAsState(targetValue = 10f)
    Column {
        Header()
        Child({offset}) 
        Footer()
    }
}

@Composable
fun Child(offset: () -> Float) {
    Box(Modifer.offset(y = offset().dp))
}
```
코드에 큰 영향을 주지 않고 읽는 시점을 제어할 수 있다. 

```kotlin
@Composable
fun Child(offset: () -> Float) {
    Box(Modifer.offset {
        IntOffset(x = 0, y = offset().toInt())
    })
}
```
컴포지션동안 실행되지 않는 Modifer 에서 이 람다를 실행한다면 이 컴포지션을 스킵할 수도 있다. 컴포지션 트리를 전혀 변경하지 않는다. 

![More performance tips for Jetpack Compose 6-32 screenshot](https://user-images.githubusercontent.com/360685/205478694-9bb288db-9a62-409b-911e-bda99476e8c6.png)

= 자주 변경되는 State 를 사용할 때는 람다 Modifier 를 선호해주자. 

![More performance tips for Jetpack Compose 7-21 screenshot](https://user-images.githubusercontent.com/360685/205478748-a35e7d8a-27f1-4870-bab1-fb9ac4f0d5eb.png)

컴포지션 트리는 컴포저블에 적용되는 컴포저블로도 build up 되는데, Modifer 는 사실상 immutable 개체다. offset 이 변경될 때마다, Modifier 가 재구성되면, 이전 값이 제거되고 새로운 것이 컴포지션 트리에 추가된다.   
> *화면을 재배치(relayout)하기 위해 recompose 할 필요는 없습니다.*

Modifer 가 실행되는 Compose 단계는?  
- Standard modiferes are <U>always</U> run during composition  
  = 람다 Modifier가 아닌 경우, 항상 Composition 으로 실행됩니다. 
- Lambda modifiers are almost certainly <U>not</U> run during composition 

추가 : [Debugging recomposition in Jetpack Compose](goo.gle/compose-debug-recomposition)

## Stability
상태가 변경되지 않았음에도 composable이 recomposition 되고 있다면?
![image](https://user-images.githubusercontent.com/360685/205479329-5cd92a71-dc61-41ad-92d1-5c79ae91900d.png)

```kotlin
@Composable
fun HomeScreen(...) {
    val contact by viewModel.contact.collectAsState()
    val selected by remember { mutableStateOf(false) } // -> true 가 되면 recomposition 시작  
    Column {
        Checkbox(selected, { selected = it })
        ContactDetails(contact)
    }
}
```
체크 박스 뿐만 아니라, 연락처가 변경되지 않아도 연락처 세부정보도 다시 호출되는 것을 볼 수 있다. 

### Recomposition
- Recomposition is the process of calling your composable functions again when inputs change. 
- When Compose recomposes based on new inputs, it only calls the funcions or lambdas that **might(?)** have changed, and skips the rest. 

컴포저블을 건너 뛰려면 변경되지 않았는지 확인해야 한다. 따라서 건너뛰어야 하는 규칙이 엄격하다. 

### Composable Functions 
- restartability : Serves as a "scope" whene recomposition can start. (recomposition 을 시작할 수 있는 지점 역할)
- Skippable : Compose is able to skip the funtion if all of the parameters are equal with their previous values. 

### Parameters 
파라미터의 안정성을 기반으로 skippable을 판단한다. 
- immutable : A type where the value of any properties will never change after the object is constructed. (`val` 타입으로 되어 있는 데이터 클래스)
- Stable : A type that is mutable, but the Compose runtime will be notified if and when any public properties change. (변경되면 compose runtime 에 알려줌)
- Unstable : None of the above. 

```kotlin
@Composable
fun ContactDetails(contact: Contact) {

}

data class Contact(var name: String) //var? - Unstable
```

```kotlin
@Composable
restartable fun ContactDetails(unstable contact: Contact) { //-> Not skippable

}
```
수정
```kotlin
data class Contact(var -> val name: String)
```

### Compose Compiler Reports 

제공해주는 것들 
- **<modulename>-class.txt** : Stability of class in this module
- **<modulename>-composables.txt** : Restartability and skippabiliy of the composables in this module. 
- **<modulename>-composables.csv** : A csv version of the above for CI or script.


```
app-composables.txt

restartable skippable
scheme("[androidx.compose.ui.Ui.Composable]")
fun ContactDetails(
    stable contact: Contact
)
```
stable 하고 skip 가능하다. 

```
app-composables.txt

restartable 
scheme("[androidx.compose.ui.Ui.Composable]")
fun ContactList(
    unstable contacts: List<Contact> // unstable?
    stable onContact: Function1<Long, Unit>
    stable modifier: Modifier? = @static Companion
)
```
= `List`, `Set`, `Map` are <U>unstable</U>

```kotlin
val name = List<String> = mutableListOf()
```

### Stabilising unstable classes 
- [KotlinX ummutable collections](https://github.com/Kotlin/kotlinx.collections.immutable)
  
  ```gradle
  dependencies {
      implementation "org.jetbrains.kotlinx:kotlinx-collections-immutable:..."
  }

  ```

  ```kotlin
  @Composable
  fun ContactList(
    contacts: ImmutableList<Contact>,
    onContactClick: (Long)-> Unit,
    modifier: Modifier = Modifier
  ) {

  }

  contacts.toImmuableList()
  ```

  모든 리스트가 **immutable** 일 필요는 없다. 성능에 문제가 있을 때 수정 방법 중 하나.

- Annotations   
  Classes from external modules are treated as **unstable**
  ```kotlin
  @Immutable // or @Stable
  data class LogEntry (
    val timeStamp: LocalDataTime, // external class
    val log: String
  )
  ```
  Incorrectly annotation a class as **stable** could cause composable to **not** recompose (잘못 추가하면 recompose 가 안 일어날 수 있음)
   
  모든 Composable 이 **skippable** 이어야 하나? No

참고 : [Jetpack Compose Stabiliy Explained](goo.gle/compose-stability-explained)

## State
### derivedStateOf {}
is used when you state or key is **changing more** than you want to update you UI (업데이트 하려는 것보다 상태나 키가 더 많이 변경될 때 사용한다.)   
= `distinctUntilChange`

![More performance tips for Jetpack Compose 16-49 screenshot](https://user-images.githubusercontent.com/360685/205480975-3c8fd90a-d17e-4eca-b503-7699f293999f.png)
![More performance tips for Jetpack Compose 17-6 screenshot](https://user-images.githubusercontent.com/360685/205480980-451f78f3-ae78-48c5-b381-c8f08b56a93e.png)

```kotlin
var firstName by remember { mutableStateOf("") }
var lastName by remeber { mutableStateOf("") }

val fullName = remeber {
    derivedStateOf { firstName + lastName }
}
```
위 경우는 상태가 변경되는 만큼 UI를 업데이트 해야 하기 때문에 `derivedStateOf` 가 무의미하다.

### remember(key) {}
State nees to changes as much as key changes 

```kotlin
val fullName = remeber(firstName, lastName) {
    expansive( firstName + lastName )
}
```
비용이 많이 두는 함수의 결과를 기억하기 위해서 remeber with key 로 설정한다. 

## reportFullyDrawn
Report to the system that your app is now fully drawn, for diagnostic and optimization purposes.  
앱이 사용할 준비가 되었음을 안드로이드에 알리는 api on Activity. 이를 통해 Android 는 실행 전에 I/O 호출을 미리 로드하여 향후 앱 시작을 최적화 할 수 있다. 

```gradle
dependencies {
    implementation "androidx.activity:activity-compose:1.7.0-alpha01"
}
```
```kotlin
@Composable
fun ReportDrawnWhenSample() {
    var contentComposed by remember { mutableStateOf(false) }

    LazyColumn {
        item {
            contentComposed = true
            Text("Hello, World")
        }
    }

    ReportDrawnWhen { contentComposed } 
}
```
`ReportDrawnWhen { contentComposed } `   
해당 조건이 참일 때 activity 에 보고한다. 리스트 구성이 완료될 때까지 기다리거나 다른 필요한 조건을 쉽게 기다릴 수 있다. 

```kotlin
@Composable
fun ReportDrawnAfterSample() {
    val lazyListState = remeberLazyListState()

    ReportDrawnAfter { //suspending version 
        lazyListState.animateScrollToItem(10)
    }

    LazyColumn(state = lazyListState) { ... }
}
```
애니메이션이 완료되거나 데이터가 로드 될때까지 기다림.

![More performance tips for Jetpack Compose 19-52 screenshot](https://user-images.githubusercontent.com/360685/205481668-5a1baa52-a3ba-4299-80d9-7026f2b882b3.png)
