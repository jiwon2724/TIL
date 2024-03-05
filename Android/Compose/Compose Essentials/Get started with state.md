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
## Composable 함수의 메모리

- Compose는 Composable 함수를 호출하여 데이터를 UI로 변환합니다.
- Composable을 실행할 때 Composable이 구축한 UI에 관한 설명을 `컴포지션`이라고 합니다.
    - 어떤 UI 요소를 그릴지, 어떤 상태에 의존하는지 등을 결정합니다.
- 상태가 변경되면 Compose는 영향을 받는 컴포저블 함수를 새 상태로 다시 실행하여 업데이트된 UI를 생성하는데, 이를 `리컴포지션`이라고 합니다.
- Compose는 데이터가 변경된 Composable 함수만 `리컴포지션`하고, 영향을 받지 않은 요소는 건너뛰도록 개별 Composable 함수에 필요한 데이터를 확인합니다.

> 컴포지션 : 컴포저블을 실행할 때 Jetpack Compose가 빌드하는 UI에 대한 설명입니다.
초기 컴포지션 : 컴포저블을 처음 실행하여 컴포지션을 생성합니다.
리컴포지션 : 데이터가 변경되면 컴포저블을 다시 실행하여 컴포저블을 업데이트합니다.
> 
- 위 처럼 프로세스를 진행하려면 **Compose가 추적할 상태**를 알아야 합니다. 그래야 업데이트를 받을 때 리컴포지션을 예약할 수 있습니다.
- Compose에는 특정 상태를 읽는 컴포저블의 리컴포지션을 예약하는 특별한 상태 추적 시스템이 있습니다.
    - 이를 통해 전체 UI가 아닌 변경해야 하는 컴포저블 함수만 리컴포지션할 수 있습니다.
    - 위 작업은 write뿐만 아니라 상태에 대한 read도 추적하여 실행됩니다.
- Comopse의 `State` 및 `MutableState`를 사용하여 Compose에서 상태를 관찰할 수 있도록 합니다.
    - 상태의 `value` 속성을 읽는 각 컴포저블을 추적하고 해당 `value`가 변경되면 리컴포지션을 트리거합니다.
    - `mutableStateOf` 함수를 사용하여 관찰 가능한 `MutableState`를 만들 수 있습니다.
        - 위 함수는 초깃값을 `State` 객체에 래핑된 매개변수로 수신한 다음, `value`의 값을 관찰 가능한 상태로 만듭니다.

> Compose에는 primitive type에 최적화된 mutableIntStateOf, mutableLongStateOf 등이 있습니다.
> 

### 상태 수정

```kotlin
@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
    val count: MutableState<Int> = mutableStateOf(0)
    println("리컴포지션!")
    Column(modifier = modifier.padding(16.dp)) {
        Text(text = "You've had ${count.value} glasses.")
        Button(onClick = { count.value++ }, Modifier.padding(top = 8.dp)) {
            Text(text = "Add One")
        }
    }
}
```

- `count`의 초깃값이 0인 `mutateStateOf` 함수를 사용하도록 `WaterCounter` 컴포저블을 업데이트합니다.
- `mutateStateOf` 가 `MutableState` 유형을 반환하므로 `value`를 업데이트하여 상태를 업데이트할 수 있고, Compose는 `value`를 읽는 이러한 함수에 리컴포지션을 트리거합니다.
- 즉, `count`가 변경되면 `count`의 `value`를 자동으로 읽는 Composable 함수의 리컴포지션이 예약됩니다. 위 코드의 경우는 버튼을 클릭할 때 마다 `WaterCounter` 컴포저블 함수는 재구성 됩니다.
- 위 방식은 리컴포지션 예약을 잘 작동합니다.
    - println("리컴포지션!") 로그 확인
- 하지만 리컴포지션이 발생하면 `count` 변수는 다시 0으로 초기화되므로 리컴포지션 간에 이 값을 유지할 방법이 필요합니다.

### 상태 수정 - remember

```kotlin
@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
    println("리컴포지션! : ")
    Column(modifier = modifier.padding(16.dp)) {
				val count: MutableState<Int> = remember { mutableStateOf(0) }
        Text(text = "You've had ${count.value} glasses.")
        Button(onClick = { count.value++ }, Modifier.padding(top = 8.dp)) {
            Text(text = "Add One")
        }
    }
}

// 위임 속성 사용
@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
   Column(modifier = modifier.padding(16.dp)) {
       var count by remember { mutableStateOf(0) }

       Text("You've had $count glasses.")
       Button(onClick = { count++ }, Modifier.padding(top = 8.dp)) {
           Text("Add one")
       }
   }
}
```

- 변경된 상태를 유지하기 위해선 Composable inline 함수인 `remember`를 사용할 수 있습니다.
- `remember`로 계산된 값은 초기 컴포지션 중에 컴포지션에 저장되고 저장된 값은 리컴포지션 간에 유지됩니다.
- 일반적으로 `remember`와 `mutableStateOf`는 Composable 함수에서 함께 사용됩니다.

## 상태 기반 UI

- Compose는 선언형 UI 프레임워크입니다.
- 상태가 변경될 때 UI 컴포넌트를 제거하거나 가시성을 변경하는 대신, 특정 상태 조건에서 UI가 어떻게 작동하는지 설명합니다.
- 리컴포지션이 호출되고 UI가 업데이트되면 컴포저블이 컴포지션을 시작하거나 종료할 수 있습니다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/17b7261d-8574-4fae-97a1-5f7367a227bc/ed055c90-22a5-47a6-a161-079c45597bb1/Untitled.png)

- 초기 컴포지션 또는 리컴포지션 동안 Composable 함수가 호출된다면, 우리는 그것이 컴포지션에 존재한다고 말합니다.
- Composable 함수가 호출되지 않는 경우 예를 들어, 함수가 if 문 내부에서 호출되었지만 조건이 충족되지 않는 경우 그것은 컴포지션에서 결석한 것으로 간주됩니다.

> UI가 사용자가 보는 것이라면, UI 상태는 앱이 사용자에게 보여주어야 한다고 지정하는 항목입니다. 동전의 양면처럼, UI는 UI 상태의 시각적 표현입니다. UI 상태에 대한 어떠한 변경도 즉시 UI에 반영됩니다.
> 

```kotlin
@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
    Column(modifier = modifier.padding(16.dp)) {
        val count: MutableState<Int> = remember { mutableStateOf(0) }

        if (count.value > 0) {
            Text("You've had ${count.value} glasses.")
        }
        
        Button(onClick = { count.value++ }, Modifier.padding(top = 8.dp), enabled = count.value < 10) {
            Text(text = "Add One")
        }
    }
}
```

- 상태는 특정 시점에 UI에 어떤 요소가 있는지를 결정합니다.
