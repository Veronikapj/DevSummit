# State holders and state production in the UI Layer

## 사전 지식 
- 비즈니스 로직 : 앱 데이터에 대한 제품 요구사항의 구현. 
- UI 로직 : 화면에 UI 상태를 표시하는 방법과 관련이 있음 

## Types of UI state 
Screen UI State : Information the app presents to the user 
UI elements'state : Internal state of a reusuable UI elements. 

![image](https://user-images.githubusercontent.com/70066242/205473209-78a9f345-4a4d-462f-95ec-16648cdbbd87.png)

viewModel
- UI  관련 로직 관리 가능 -> viewModelScope 를 통해 vm 라이프 사이클에 맞게 적용 가능. 또한 vm을 사용하면서 configuration change에 대응이 가능해짐 -> activity, fragment 는 각각의 life cycle이 있어 거기에 종속이 되지만, vm을 사용함으로써 UI 관련 데이터 관리가 좀 더 용이해짐. 
- viewModel은 UI를 몰라야 한다. 
- context와 같은 것을 잡고(hold) 있지 않아야 한다. 이를 위반하면 memory leak이 유발될 수 있음. viewModel은 view 보다 더 오래 살아있어야 한다. 

## 활용

![image](https://user-images.githubusercontent.com/70066242/205473106-9f93424e-2aaf-48eb-90a5-34c6c45fc9c7.png)

▲ 기본적인 viewModel 활용 

![image](https://user-images.githubusercontent.com/70066242/205473130-ede1820d-4013-48de-a848-883f8a360b2f.png)

▲ 코드가 좀 더 추가된 viewModel 

![image](https://user-images.githubusercontent.com/70066242/205473162-d159428d-4fd4-4126-8216-a558c02978c1.png)

▲ 상태를 관리해주는 sealed interface

![image](https://user-images.githubusercontent.com/70066242/205473177-c6c94157-7aa9-4e14-a31c-3f9be9b15729.png)

▲ view에서 구현 

## 추가 공부

enum / sealed class 차이 

**In a \*sealed class\*, we can simply add multiple custom constructors depending on what we need.**

**In an \*enum class\*, however, we can’t define different functions in each \*enum\* constant.**

외부의 다른 class들은 이 sealed class를 상속받을 수 없도록 정의되어 있습니다. 이러한 봉인된 구조로 인해 컴파일 타임에 코드를 작성하며, 서브클래스들에 접근해서 사용이 가능하다. 

생성자도 각각의 특징에 따라 다양하게 가져갈 수 있다. 

sealed interface : sealed class의 생성자를 가지고 있지 않은 interface 버전. (생성자가 필요하지 않을 때 사용 )



Enum class : 생성자의 형태가 모두 동일해야 한다. (sealed class가 좀 더 유연..?)



상황에 따라 맞춰서 enum / sealed class 사용! 
