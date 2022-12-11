# Deep dive into Wear OS app architecture

피트니스 앱을 보며 wear OS 이해

<img width="745" alt="스크린샷 2022-12-11 오후 8 32 12" src="https://user-images.githubusercontent.com/70066242/206901164-ce125ee5-b28e-45c6-b19b-a7999255c102.png">


## 모듈화 장점

<img width="704" alt="스크린샷 2022-12-11 오후 8 32 25" src="https://user-images.githubusercontent.com/70066242/206901184-0886888e-f4ea-4bae-bf88-70d37ed26183.png">
Scalability : 관심사 분리를 수용하며, 사람들이 변화를 만들 수 있도록 권한을 부여

Encapsulation : 코드를 분리함으로써 읽기 쉬운 코드, 테스트의 편리성 상승 및 유지 보수가 쉬워짐 

Reusability : multiple 앱의 동일한 기반에서 여러 앱을 빌드가 가능해짐 

<img width="742" alt="스크린샷 2022-12-11 오후 8 32 38" src="https://user-images.githubusercontent.com/70066242/206901205-33cc7f00-f55d-4b4a-8cc9-9721eecd10e9.png">

기존의 app:mobile 모듈에서 app:wear 모듈만 새로 만든 후, 나머지 모듈은 가져오는 방식으로 개발을 진행할 수 있다.


## Data layer

remote / local 통신 - 아키텍처가 app:mobile과 동일하기에 재사용 가능. UI는 구성이 다르기에 불가능!
[운동 기간 / 센서 및 운동 강도 기록]

## Domain layer

<img width="454" alt="스크린샷 2022-12-11 오후 8 32 54" src="https://user-images.githubusercontent.com/70066242/206901222-3a5dc874-2aea-4f81-b577-7dbd0043dce8.png">


<img width="434" alt="스크린샷 2022-12-11 오후 8 33 00" src="https://user-images.githubusercontent.com/70066242/206901221-8ccd8898-0b3a-45a8-a40c-2d5f515f20a0.png">

usecase는 single responsibility 를 갖는다.(SOLID)

최근 일주일 간에 정보도 갖고 싶었다. 그렇기에 usecase를 중간에 넣어주게 되었습니다.

<img width="720" alt="스크린샷 2022-12-11 오후 8 33 17" src="https://user-images.githubusercontent.com/70066242/206901218-462a8ef7-80bf-40a1-93a8-a504d806b1b9.png">


## UI Layer

Horologist: wear OS 작업 도와주는 Lib. Horologist는 Google에서 제공하는 라이브러리 그룹으로, 개발자에게 일반적으로 필요하지만 아직 Jetpack에서 사용할 수 없는 기능을 제공하여 Wear OS 개발자를 돕는 것을 목표로 합니다.


Horologis 깃허브 주소:
https://github.com/google/horologist
