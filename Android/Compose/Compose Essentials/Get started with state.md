# **Get started with state**

> 본 포스팅은 아래 Compose essentials codelab을 학습하고 정리한 포스팅 입니다.
> 

[Compose essentials - Get started with state](https://developer.android.com/codelabs/jetpack-compose-state?continue=https%3A%2F%2Fdeveloper.android.com%2Fcourses%2Fpathways%2Fjetpack-compose-for-android-developers-1%3Fhl%3Dko%23codelab-https%3A%2F%2Fdeveloper.android.com%2Fcodelabs%2Fjetpack-compose-state&_gl=1*1di3hd5*_up*MQ..*_ga*NDg0NTI2MTc5LjE3MDk1NTgzODQ.*_ga_6HH9YJMN9M*MTcwOTU1ODM4NC4xLjAuMTcwOTU1ODM4NC4wLjAuMA..#0)

- Compose에서 상태(State)를 사용하는 것과 관련된 핵심 개념을 설명합니다.
    - 앱의 상태(State)에 따라 UI에 표시되는 항목이 결정되는 방식
    - 상태가 변경될 때 다양한 API를 사용해 Compose에서 UI를 업데이트하는 방법
    - Composable 함수의 구조를 최적화하는 방법
    - Compose 환경에서 ViewModel을 사용하는 방법

## Compose에서의 상태

- 앱의 상태(State)는 시간이 지남에 따라 변할 수 있는 값입니다.
    - 이는 매우 광범위하며 Room Database부터 클래스, 변수까지 모든 항목이 포함됩니다.
- 모든 Android 앱에서는 사용자에게 상태가 표시됩니다. 상태의 몇가지 예시는 다음과 같습니다.
    - 채팅 앱에서 가장 최근에 수신된 메시지
    - 사용자의 프로필 사진
    - 목록의 스크롤 위치

> 상태에 따라 특정 시점에 UI에 표시되는 항목이 결정됩니다.
> 

### 하루 동안 마신 물잔 개수를 계산하는 기능 만들기

- 첫 번째로 만드는 기능은 하루 동안 마신 물잔 개수를 계산하는 `WaterCount`입니다.
- Composable 함수를 사용하여 `count`라는 값에 저장해야합니다.

```kotlin
@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
    val count = 0
    Text(
        text = "You've had $count glasses.",
        modifier = modifier.padding(16.dp)
    )
}
```

- `WaterCount`함수의 상태는 `count`변수입니다. 그러나 정적 상태(`val`)는 수정할 수 없기 때문에 유용하지 않습니다.
- Button을 추가하여 개수를 늘리고 하루 동안 마신 물잔 개수를 추적해 봅시다.

## Compose의 이벤트

- 상태가 수정되도록 하는 작업을 `이벤트` 라고 부릅니다.
- 상태는 시간이 지남에 따라 변합니다. Android 앱에서는 `이벤트`에 대한 응답으로 상태가 업데이트됩니다.
- `이벤트`는 앱 외부 또는 내부에서 생성되는 입력입니다. 예를 들면
    - 버튼 누르기 등으로 UI와 상호작용하는 유저
    - 기타 요인(ex : 새 값을 전송하는 센서 또는 네트워크 응답)
- 상태(State)로 UI에 표시될 항목에 관한 설명이 제공되고, 이벤트라는 메커니즘을 통해 상태가 변경되고 UI도 변경됩니다.

> 상태는 존재하고, 이벤트는 발생합니다.
> 

![스크린샷 2024-03-04 오후 10 55 48](https://github.com/jiwon2724/TIL/assets/70135188/19cdb9c4-daa3-4d8f-b7ab-891ac872c12c)


- Event : 사용자 또는 프로그램의 다른 부분에 의해 생성됩니다.
- Update State : 이벤트 핸들러가 UI에서 사용하는 상태를 변경합니다.
- Display State : 새로운 상태를 표시하도록 UI가 업데이트됩니다.

### 상태를 수정할 수 있도록 버튼 추가하기

- 이제 물잔을 더 추가하여 상태를 수정할 수 있도록 버튼을 추가해봅시다.

```kotlin
@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
    var count = 0
    Column(modifier = modifier.padding(16.dp)) {
        Text(text = "You've had $count glasses.")
        Button(onClick = { count++ }, Modifier.padding(top = 8.dp)) {
            Text(text = "Add One")
        }
    }
}
```

- 버튼을 클릭해도 아무 일도 일어나지 않습니다. Compose에서는 이 값을 `상태 변경(state change)`으로 감지하지 않기 때문입니다.
- 상태가 변경될 때 Compose에 화면을 다시 그려야 한다고 알리지 않았기 때문입니다.
    - 즉, 구성 가능한 함수(Composable)를 재구성하지 않았습니다.
