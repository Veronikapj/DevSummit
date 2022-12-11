## Compose Modifiers deep dive

"내용 요약 : Compose Modifier 도입기 및 변천사 "

Modifier란?
compose 를 사용하면서 Modifier가 등장하였다. Modifier(padding, backgroundColor..  )는 Compose의 UI를 수정 및 행동 추가(클릭과 같은)와 같은 기능을 수행할 수 있다. 

```
// multiple children 
Clickable() {
  Row(
    Content()
    Content()
    )
}
```
modifier 없이 사용한 경우 

중복 코드(여러 composable 발생)에서 문제가 발생 => 우선 Row 혹은 Colum으로 처리 => 사용자 혼동 

코드에서 혼동을 줄 수 있기에 modifier로 따로 빼서 사용 => 명확한 구분이 가능해짐 

여러 child 생성 허용 => 복잡하며, 상태 관리를 비롯하여 다양한 기능을 사용하기에 효율성이 떨어짐 

```
@Composable fun Example1() {
	Row(Modifier.padding(10.dp).background(Color.Blue).clickable{...}
	) {
	  Content()
	  Contene()
	}
}
```
각 레이아웃별로 상태가 존재해야 하기에 Modifier.composed api 사용 

람다식 사용 Modifier 상태 관리 가능 

```
@Composable fun materialize(modifier: Modifier): Modifier

fun Modifier.composed(
	factory: @Composable.() -> Modifier
	): Modifier 
```
여기서 상태 관리 관련해서 문제 발생 => 위의 방법을 통해 상태 관리 가능해짐 

materialize라는 내부 함수 사용하여 레이아웃 별로 호출이 가능해 짐. 

Modifier.clickable은 리턴 값을 갖기에 Modifier.composed도 리턴 값을 갖는다. => 그렇기에 Modifier.composed 사용한 Modifier는 skip 불가능! / 람다 캐시도 불가능 -> recomposition이 발생하면 어떤 것이 변하였는지 확인이 불가능 해서 비용 증가. 


### 해결을 위해 Modifier 내부에 "@Experimental abstract class Node { ... }" 추가 

이를 통해 recomposition이 발생하여도 쉽게 비교가 가능해지고 비용이 감소하게 되었음. 



### 결과

- Less Allocations : 구체화 하는 동안 Modifier chain을 재구성할 필요가 없고 chain 자체가 더 짧으며 재구성할 때마다 레이아웃에 엔티티를 할당할 필요가 없어짐
- Less Composition : 상태를 remember하거나 변경된 매개변수를 proxy하기 위해 snapshot state를 사용할 필요가 없으므로 Modifier와 같이 overhead가 감소
- Smaller Tree : clickable과 같은 Modifier는 16개의 노드를 생성하곤 했습니다. 이제 이 대신 단일 노드만 생성

