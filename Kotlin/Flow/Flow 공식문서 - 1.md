### 비동기 Flow

- 일시 중단 함수들은 비동기적으로 단일 값을 반환함.
- 비동기적으로 계산한 복수의 값들을 반환하기 위해  Kotlin의 Flows가 등장함.

```kotlin
Kotlun의 Flow는 순차적으로 값을 내보내고 정상적으로 또는 예외로 완료되는 비동기적인 데이터 스트림입니다.
```

### 복수의 값들 표현

- Collection을 사용하여 Kotlin에서 복수의 값을 나타낼 수 있음.
- 세 개의 숫자 목록을 반환한 다음 forEach를 사용하여 모두 출력하는 간단한 함수를 가질 수 있음.

```kotlin
fun simple(): List<Int> = listOf(1, 2, 3)
 
fun main() {
    simple().forEach { value -> println(value) } 
}

실행결과
1
2
3
```

### Sequences

- CPU 리소스를 사용하면서 블로킹 하는 코드로 숫자에 대한 연산을 한다면 `Sequence` 를 사용해 숫자를 나타낼 수 있음.

```kotlin
fun simple(): Sequence<Int> = sequence { // sequence builder
    for (i in 1..3) {
        Thread.sleep(100) // pretend we are computing it
        yield(i) // yield next value
    }
}

fun main() {
    simple().forEach { value -> println(value) } 
}
```

- 위 예제에선 `Sequence` 를 사용 안해도 됨.
    - 중간 연산이 없음.

### 일시중단 함수들

- 위 `Sequence` 예제는 메인 스레드를 블로킹함.
- 스레드를 블로킹 시키지 않고 비동기 코드에 의해 계산된다면, `suspend` 키워드를 붙여  수정할 수 있음.

```kotlin
import kotlinx.coroutines.*

suspend fun simple(): List<Int> {
    delay(1000)
    return listOf(1, 2, 3)
}

fun main() = runBlocking<Unit> {
    simple().forEach { value -> println(value) }
}
```

### Flows

- 결과 타입으로 `List<XXX>`를 사용하면 한 번에 모든 값을 반환해야함.
- 동기적으로 계산된 값은 `Sequence<XXX>`를 사용해 나타냈던 것처럼
- 비동기적으로 계산되는 값들을 스트림으로 나타내려면 `Flow<XXX>` 타입을 사용할 수 있음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow {
    for(i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    launch {
        repeat(3) {
            println("I'm not blocked ${it+1}")
            delay(100)
        }
    }
    simple().collect { value -> println(value) }
}
```

- `flow { .. }` 는 플로우를 만드는 플로우 빌더 함수임.
    - 해당 블록의 내부 코드들은 일시 중단 될 수 있음.
- `emit` 함수를 사용해 flow에서 값들이 방출됨.
- `collect` 함수를 사용해 flow로부터 값들을 수집함.

### Flows는 차갑다

- Flow는 cold stream이다.
    - 이로인해 flow 빌더 내부의 코드는 flow가 collect되기 전까지 실행되지 않음.

```kotlin
cold stream이란?
소비하기 시작하면 데이터를 발행하는 구조. 즉, collect를 시작해야 emit이 됨.
한명의 구독자만 존재하며, 새로운 구독자가 있을 경우 플로우가 새로 시작됨.
```

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow {
    println("Flow started")
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    println("Calling simple function...")
    val flow = simple()
    println("Calling collect...")
    flow.collect { value -> println(value) }
    println("Calling collect again...")
    flow.collect { value -> println(value) }
}

실행결과
Calling simple function...
Calling collect...
Flow started
1
2
3
Calling collect again...
Flow started
1
2
3
```

- simple 함수 호출 그 자체는 바로 반환되며, 어떤 것도 기다리지 않음.
    - flow 함수는 `Flow` 객체를 즉시 반환함.
    - flow를 반환하는 함수가 `suspend`로 표시되지 않는 이유임.

### Flow 취소 기초

- `Flow`는 코루틴의 기본 협력적인 취소를 따름.
- 일반적으로 취소 가능한 일시 중단 함수에서 `Flow`가 일시 중단될 때 `Flow`로 부터 값을 수집하는 것이 취소될 수 있음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    withTimeoutOrNull(250) {
        simple().collect { value -> println(value) }
    }
    println("Done")
}

실행결과
Emitting 1
1
Emitting 2
2
Done
```

### Flow 빌더

- `flow { … }`는 `Flow`의 가장 기본적인 빌더임.
- `flowOf` 빌더는 정해진 값의 세트를 방출하는 Flow를 정의함.
- 다양한 `Collection`들과 `Sequence`들은 `.asFlow()` 확장 함수를 사용해 Flow로 변환이 가능함.

```kotlin
val flow = flowOf(1, 2, 3)
val arrange = (4..6)

fun main() = runBlocking<Unit> {
    flow.collect { println(it) }
    arrange.asFlow().collect { println(it) }
    println("Done")
}

실행결과
1
2
3
4
5
6
Done
```

### Flow 중간 연산자

- Flow들은 `Collection` 들과, `Sequence` 와 같이 연산자를 이용해 변환될 수 있음.
- 중간 연산자는 업 스트림 `Flow`에 적용되어 다운 스트림 `Flow`를 반환함.
    - 이 자체로 일시 중단 함수가 아님. 빠르게 작동해 새로운 `Flow`를 반환함.
- `map` , `filter` 등 이러한 연산자들과 `Sequence` 의 차이점은 코드 블록 내부에서 일시 중단 함수를 호출할 수 있음.

```kotlin
중간 연산자와 종료 연산자란?

중간 연산자 
원래의 Flow를 변환하여 새로은 Flow를 생성하는 연산자를 의미함.
ex) map, filter, transform, take 등

종료 연산자 
Flow에서 데이터를 수집하고 처리하는 역할.
ex) collect, toList, first 등
```

```kotlin
업 스트림, 다운 스트림이란?

업 스트림 : Flow에서 데이터를 생성하거나, 변형하거나, 필터링하는 등의 중간 연산을 수행하는 부분.
다운 스트림 : 업 스트림에서 발행된 데이터를 수집(종료 연산자를 사용)하여 처리하는 부분. 

```

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

suspend fun performRequest(request: Int): String {
    delay(1000) 
    return "response $request"
}

fun main() = runBlocking<Unit> {
    (1..3).asFlow()
        .map { request -> performRequest(request) } // 업 스트림. 중간 연산을 실행함. 해당 부분의 데이터를 수집되지 않음.
        .collect { response -> println(response) } // 다운 스트림. 업 스트림에서 발행한 데이터를 수집하여 처리함.
}
```
