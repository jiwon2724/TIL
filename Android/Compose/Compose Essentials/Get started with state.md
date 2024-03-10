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

![스크린샷 2024-03-05 오후 10 48 23](https://github.com/jiwon2724/TIL/assets/70135188/d903dc22-ad34-4a8c-9bd9-8e7b07b42ec7)


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

## 컴포지션의 Remember

- `remember`는 컴포지션에 객체를 저장하고, `remember`가 호출되는 소스 위치가 리컴포지션 중에 다시 호출되지 않으면 객체를 삭제합니다.

```kotlin
@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
   Column(modifier = modifier.padding(16.dp)) {
       var count by remember { mutableStateOf(0) }
       if (count > 0) {
           var showTask by remember { mutableStateOf(true) }
           if (showTask) {
               WellnessTaskItem(
                   onClose = { showTask = false },
                   taskName = "Have you taken your 15 minute walk today?"
               )
           }
           Text("You've had $count glasses.")
       }

       Row(Modifier.padding(top = 8.dp)) {
           Button(onClick = { count++ }, enabled = count < 10) {
               Text("Add one")
           }
           Button(
               onClick = { count = 0 },
               Modifier.padding(start = 8.dp)) {
                   Text("Clear water count")
           }
       }
   }
}
```

- `count`, `showTastk` 는 remember 변수입니다.
- Add one 버튼을 누르면 `count`가 증가하고 리컴포지션이 발생합니다.
    - `WellnessTaskItem` 및 Text 컴포저블의 count가 표시되기 시작합니다.

![스크린샷 2024-03-06 오후 10 50 47](https://github.com/jiwon2724/TIL/assets/70135188/65bbf76f-4459-4eab-91a3-ff1fc57b7edc)


- `WellnessTaskItem` 의 구성요소의 X를 누릅니다. 이때 리컴포지션이 발생하고, `showTask`가 false이므로 `WellnessTaskItem` 은 더 이상 표시되지 않습니다.

![스크린샷 2024-03-06 오후 10 53 09](https://github.com/jiwon2724/TIL/assets/70135188/658725aa-9a38-4ef0-b6cd-24c398200ab6)


## Compose에서 상태 복원

- 구성 변경시 저장된 상태는 삭제됩니다.
- `remember`를 사용하면 리컴포지션 간에 상태를 유지하는데 도움되지만, 구성 변경 간에는 유지되지 않습니다.
- 이를 위해서 `remember`대신 `rememberSaveable`을 사용해야 합니다.
- `rememberSaveable` 은 `Bundle`에 저장할 수 있는 모든 값을 자동으로 저장합니다.

```kotlin
@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
        ...
        var count by rememberSaveable { mutableStateOf(0) }
        ...
}
```
## 상태 호이스팅

- `remember`를 사용하여 객체를 저장하는 컴포저블 함수에는 내부 상태가 포함되며 이는 컴포저블 함수를 Stateful하게 만듭니다.
    - 상태를 갖는 컴포저블은 **Stateful 컴포저블**이라고 합니다.
- 이는 **호출자가 상태를 제어할 필요가 없고 상태를 직접 관리하지 않아도 상태를 사용할 수 있는 경우에 유용**합니다.
    - 하지만 내부 상태를 갖는 컴포저블은 재사용 가능성이 적고 테스트하기가 더 어려운 경향이 있습니다.
- 상태를 보유하지 않은 컴포저블을 **Stateless 컴포저블**이라고 합니다. 상태 호이스팅을 사용하면 Stateless 컴포저블을 쉽게 만들 수 있습니다.
- 즉, 상태 호이스팅은 컴포저블을 Stateless로 만들기 위해 상태를 컴포저블의 호출자로 옮기는 패턴입니다.
- 상태 호이스팅을 위한 일반적인 패턴은 상태 변수를 다음 두 개의 매개변수로 바꾸는 것입니다.
    - `value: T` : 표시할 현재 값입니다.
    - `onValueChange: (T) → Unit` : 값이 새 값 T로 변경되도록 요청하는 이벤트입니다.
- 위 값은 수정할 수 있는 모든 상태를 나타냅니다.

> 상태가 내려가고 이벤트가 올라가는 패턴을 단방향 데이터 흐름(UDF)이라고하며, 상태 호이스팅은 이 아키텍처를 Compose에서 구현하는 방법입니다.
> 
- 이러한 방식으로 끌어올린 상태에는 중요한 속성이 몇 가지 있습니다.
    - 단일 소스 저장소 : 상태를 복제하는 대신 옮겼기 때문에 소스 저장소가 하나만 있습니다.
        - 버그 방지에 도움이 됩니다.
    - 공유 가능 : 끌어올린 상태를 여러 컴포저블과 공유할 수 있습니다.
    - 분리(Decoupling) : Stateless 함수의 상태는 어디에든(ex : ViewModel) 저장할 수 있습니다.

> Stateless : 상태를 소유하지 않는 컴포저블입니다. 즉, 새 상태를 보유하거나 정의하거나 수정하지 않습니다.

Stateful : 시간이 지남에 따라 변할 수 있는 상태를 소유하는 컴포저블입니다.

컴포저블이 가능한 적게 상태를 소유하고 적절한 경우 컴포저블 API에 상태를 노출하여 끌어올릴 수 있도록(호이스팅) 컴포저블을 디자인해야 합니다.
> 

### Stateful, Stateless 컴포저블 만들기

```kotlin
@Composable
fun StatelessCounter(
    count: Int, // value
    onIncrement: () -> Unit, // onValueChange
    modifier: Modifier = Modifier
) {
    Column(modifier = modifier.padding(16.dp)) {
        if (count > 0) {
            Text("You've had $count glasses.")
        }
        Button(
            onClick = onIncrement,
            enabled = count < 10,
            modifier = Modifier.padding(top = 8.dp)
        ) {
            Text("Add one")
        }
    }
}
```

- `StatelessCount`의 역할은 `count`를 표시하고 `count`를 늘릴 때 함수를 호출합니다.
- `count` 의 상태와 `onIncrement` 람다를 전달합니다.

```kotlin
@Composable
fun StatefulCounter(modifier: Modifier = Modifier) {
    var count by rememberSaveable { mutableStateOf(0) }
    StatelessCounter(
        count = count,
        onIncrement = { count++ },
        modifier = modifier
    )
}
```

- `StatefulCounter` 는 상태를 소유합니다. count의 상태를 보유하고 `StatelessCounter` 함수를 호출할 때 이 상태를 수정합니다.

> 상태 호이스팅을 사용할 때 이동 위치를 쉽게 파악할 수 있는 세 가지 규칙이 있습니다.

1. 상태를 사용하는 모든 컴포저블의 가장 낮은 공통 부모(읽기)로 상태를 올려야 합니다.

2. 상태는 최소한 변경될 수 있는 가장 높은 수준으로 올려야 합니다(쓰기).

3. 동일한 이벤트에 대한 응답으로 두 상태가 변경되는 경우 동일한 레벨로 올려야 합니다.

상태를 충분히 높은 수준으로 끌어올리지 않으면, UDF패턴을 따르기가 어렵거나 불가능 할 수 있습니다.
> 

### Stateless 컴포저블 재사용

```kotlin
@Composable
fun StatefulCounter() {
    var waterCount by remember { mutableStateOf(0) }
    var juiceCount by remember { mutableStateOf(0) }

    StatelessCounter(waterCount, { waterCount++ })
    StatelessCounter(juiceCount, { juiceCount++ })
}
```

- waterCount나 juiceCount의 상태가 변경될 때 리컴포지션이 일어납니다.
- 리컴포지션 중에 Compose는 해당 상태를 읽는 함수만 식별하고 변경된 상태를 사용하는 함수의 리컴포지션만 트리거합니다.
    - ex) `juiceCount++`가 호출되면 `StatelessCounter(juiceCount, { juiceCount++ })`만 리컴포지션 됩니다.

> 호이스팅된 상태는 공유할 수 있으므로 불필요한 리컴포지션을 방지하고 재사용성을 높이려면 컴포저블에 필요한 상태만 전달해야 합니다.

핵심 사항 : 컴포저블 디자인 권장사항은 필요한 매개변수만 전달하는 것입니다.
> 

## 목록으로 작업하기

```kotlin
// Stateless
@Composable
fun WellnessTaskItem(
    taskName: String,
    checked: Boolean,
    onCheckedChange: (Boolean) -> Unit,
    onClose: () -> Unit,
    modifier: Modifier = Modifier
) {
    Row(
        modifier = modifier,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(
            modifier = Modifier
                .weight(1f)
                .padding(start = 16.dp),
            text = taskName
        )
        Checkbox(
            checked = checked,
            onCheckedChange = onCheckedChange
        )
        IconButton(onClick = onClose) {
            Icon(Icons.Filled.Close, contentDescription = "Close")
        }
    }
}

// Statefult
@Composable
fun WellnessTaskItem(taskName: String, modifier: Modifier = Modifier) {
    var checkedState by remember { mutableStateOf(false) }

    WellnessTaskItem(
        taskName = taskName,
        checked = checkedState,
        onCheckedChange = { newValue -> checkedState = newValue },
        onClose = {}, // we will implement this later!
        modifier = modifier,
    )
}

data class WellnessTask(
    val id: Int,
    val label: String,
)

@Composable
fun WellnessTasksList(
    modifier: Modifier = Modifier,
    list: List<WellnessTask> = remember { getWellnessTasks() }
) {
    LazyColumn(
        modifier = modifier
    ) {
        items(
            items = list,
        ) { task ->
            WellnessTaskItem(
                taskName = task.label,
            )
        }
    }
}

private fun getWellnessTasks() = List(30) { i -> WellnessTask(i, "Task # $i") }

@Composable
fun WellnessScreen(modifier: Modifier = Modifier) {
   Column(modifier = modifier) {
       StatefulCounter()
       WellnessTasksList()
   }
}

```

- LazyLayout을 사용해서 Task 목록을 만들었습니다.

### LazyList에서 항목(item) 상태 복원

```kotlin
@Composable
fun WellnessTaskItem(
    taskName: String,
    modifier: Modifier = Modifier
) {
    var checkedState by remember { mutableStateOf(false) }

    WellnessTaskItem(
        taskName = taskName,
        checked = checkedState,
        onCheckedChange = { newValue -> checkedState = newValue },
        onClose = {}, // we will implement this later!
        modifier = modifier,
    )
}
```

- `checkedState` 가 변경되면 `WellnessTaskItem` 의 인스턴스만 재구성되며 `LazyColumn`의 모든 `WellnessTaskItem` 인스턴스가 재구성되는 것은 아닙니다.
- 항목이 컴포지션을 종료하면 기억된 상태가 삭제된다는 문제가 있습니다.
- `LazyColumn` 에 있는 항목의 경우 스크롤하면서 항목을 지나치면 컴포지션을 완전히 종료하므로 체크된 항목의 선택이 해제되어 있습니다.
- 위 경우 다시 `rememberSaveable`을 사용하면 됩니다.

```kotlin
var checkedState by rememberSaveable { mutableStateOf(false) }
```

## 관찰 가능한 MutableList

- 리스트에서 Task를 삭제하는 동작을 추가해봅시다. 먼저 목록을 변경 가능한 목록으로 만들어야 합니다.
- `ArrayList<T>` 또는 `mutableListOf` 를 사용하면 작동하지 않습니다.
    - 위 유형은 항목이 변경되었고, UI의 리컴포지션을 예약한다고 Compose에 알리지 않습니다.
- Compose에서 관찰할 수 있는 `MutableList` 인스턴스를 만들어야 합니다.
    - 이 구조를 사용하면 Compose가 하목이 추가되거나 목록에서 삭제될 때 변경사항을 추적하여 리컴포지션 할 수 있습니다.

```kotlin
@Composable
fun WellnessScreen(modifier: Modifier = Modifier) {
   Column(modifier = modifier) {
       StatefulCounter()

       val list = remember { getWellnessTasks().toMutableStateList() }
       WellnessTasksList(list = list, onCloseTask = { task -> list.remove(task) })
   }
}

private fun getWellnessTasks() = List(30) { i -> WellnessTask(i, "Task # $i") }
```

> mutableStateListOf를 사용하여 목록(List)을 구현할 수 있습니다. 그러나 이를 사용하는 방식으로 인해 예기치 않은 리컴포지션이 발생하고 UI 성능이 최적화되지 않을 수 있습니다.

목록을 정의하고 작업을 다른 작업에 추가하면 모든 리컴포지션에 중복된 항목이 추가됩니다.
> 
> 
> `// Don't do this!`
> 
> `val list = remember { mutableStateListOf<WellnessTask>()` `}`
> 
> `list.addAll(getWellnessTasks())`
> 
> 단일 작업으로 초깃값을 사용하여 만든 후, 다음과 같이 remember 함수에 전달합니다.
> 
> `// Do this instead. Don't need to copy`
> 
> `val list = remember {`
> 
> `mutableStateListOf<WellnessTask>().apply {` `addAll(getWellnessTasks()) }`
> 
> `}`
> 

## ViewModel의 상태

- UI 상태는 화면에 표시할 내용을 설명하지만 앱의 로직은 앱의 동작 방식을 설명하고 상태 변경에 반응해야 합니다.
- 로직 유형에는 두가지가 있습니다.
    - UI 로직 : 화면에 상태 변경을 표시하는 방법과 관련이 있습니다.
        - ex) 탐색 로직 또는 스낵바 표시
    - 비즈니스 로직 : 상태 변경 시 실행할 작업입니다. 대개 비즈니스 레이어나 데이터 영역에 배치되고 UI 레이어에는 배치되지 않습니다.
        - ex) 결제하기, 사용자 환경설정 저장

> ViewModel은 컴포지션의 일부가 아닙니다. 따라서 메모리 누수가 발생할 수 있으므로 컴포저블에서 만든 상태를 보유해서는 안됩니다.
> 

### ViewModel로 마이그레이션

- 구성 가능한 함수에서 상태를 직접 관리하는 방법을 알아봤지만, UI 로직과 비즈니스 로직을 UI 상태와 분리하여 ViewModel로 옮기는 것이 좋습니다.

```kotlin
class WellnessViewModel : ViewModel() {
    private val _task = getWellnessTasks().toMutableStateList()
    val task: List<WellnessTask>
        get() = _task

    fun remove(item: WellnessTask) {
        _task.remove(item)
    }
}

private fun getWellnessTasks() = List(30) { i -> WellnessTask(i, "Task # $i") }
```

- `viewModel()` 함수를 호출하여 컴포저블에서 ViewModel을 참조할 수 있습니다.

```kotlin
implementation("androidx.lifecycle:lifecycle-viewmodel-compose:{latest_version}")
```

- 선택된 상태와 로직을 ViewModel로 이전하는 것이 좋습니다. 이렇게 하면 모든 상태가 ViewModel에서 관리되므로 코드가 더 간단해지고 테스트하기 쉬워집니다.

> ViewModel 인스턴스를 다른 컴포저블에 전달하는 것은 좋지 않습니다. 필요한 데이터와 필수 로직을 실행하는 함수만 매개변수로 전달해야 합니다.

<img width="686" alt="스크린샷 2024-03-09 오후 4 02 10" src="https://github.com/jiwon2724/TIL/assets/70135188/6ac35e82-3220-41f8-b83b-5cf08c79c375">


## 궁금한 부분

- 컴포지션은 컴포저블 함수의 로직이라고 봐도 될까요?
    - ex) 컴포저블 함수 안에 다양한 컴포저블 함수들이 있고, 상태를 변경하는 코드가 있는 경우에 해당 함수를 호출한다면 이것이 컴포지션?
    - 아니면 컴포지션이라는 다른 무엇이 있을까요? ex) XML에서 View를 그리는 과정 등등..
> 컴포지션은 컴포저블 함수가 UI 트리를 구성하는 것 입니다.

- remember를 사용할 땐 값을 저장 후 리컴포지션이 일어날까요?
    - ex) count.value++
    - remember { mutableStateOf(0) }
    - 위 같은 경우에 count를 1증가 시키고 remember에 저장한 후 리컴포지션이 일어나는지!?
> rememeber는 지정한 객체를 유지하고 싶을 때 사용 위 내용이 맞음.

- 위임 속성을 사용해서 remember를 사용하는 문법은 팀바팀 회바회?
- 상태 기반 UI 섹션이 이해가 잘 안가요 ㅠ
- 호이스팅은 그냥 상태를 매개변수로 넘겨줘서 사용하는 느낌일까요?
    - “상태를 끌어올린다”의 정의가 아리송합니다
- 호이스팅의 이동위치를 파악하는 세 가지 규칙 중 공통적인 부분은 사용하려는 상태에 맞게 잘 사용해라 느낌?

> 최상위 부모에서 계속 아래단으로, 변경과 공급은 필요한 곳 까지 내려주는
