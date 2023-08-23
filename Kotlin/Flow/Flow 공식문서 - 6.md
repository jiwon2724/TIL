### 선언적인 처리

- 선언적으로 접근하면, Flow는 Flow의 수집이 완료되었을 때 실행되는 `onCompletion` 중간 연산자가 있음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow {
    emit(1)
    throw RuntimeException()
}

fun main() = runBlocking {
    simple()
        .onCompletion { cause -> if (cause != null) println("Flow completed exceptionally") }
        .catch { cause -> println("Caught exception $cause") }
        .collect { value -> println(value) }
}

실행결과
1
Flow completed exceptionally
Caught exception java.lang.RuntimeException
```

- `onCompletion` 연산자는 `catch` 와 다르게 예외를 처리하지 않음.
- 예외는 여전히 다운스트림으로 흐름.
    - `onCompletion` 연산자로 전달되며, 이후에 `catch` 연산자를 사용해 처리될 수 있음.

### 성공적인 완료

- `catch` 연산자와 또 다른 점은 `onCompletion`는 모든 예외를 볼 수 있고, 업스트림 `Flow`가 취소나 실패 없이 성공적으로 완료되었을 때 null을 수신함.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking {
    simple()
        .onCompletion { cause -> println("Flow completed with $cause") }
        .collect { value -> println(value) }
}

실행결과
1
2
3
Flow completed with null
```

- 위 예제에선 취소나 실패 없이 성공적으로 완료되었으므로, null을 수신함.

```kotlin
fun main() = runBlocking {
    simple()
        .onCompletion { cause -> println("Flow completed with $cause") }
        .collect { value ->
            check(value <= 1) { "Collected $value" }
            println(value)
        }
}

실행결과
1
Flow completed with java.lang.IllegalStateException: Collected 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
...
```

- 다운 스트림 예외로 인해 `Flow`가 중단되었으므로, `onCompletion` 에 들어온 예외는 null이 아님.

### 명령적으로 다루기 vs 선언적으로 다루기

- 명령적 방식과 선언적 방식은 두 접근 방식 모두 유효함.
- 이는 선호도와 코드 스타일에 따라 선택되어야 함.

### Flow 실행하기

- 비동기 이벤트를 표현하기 위해 Flow를 사용하기 쉬움.
    - 들어오는 이벤트에 대한 반응을 코드로 등록하고 이후의 작업을 계속해서 수행하도록 하는 `addEventListener` 함수와 비슷한 역할을 하는 것이 필요함.
    - `onEach` 연산자가 그런 역할을 해줌.
        - 이는 중간 연산자임.
        - 터미널(종단) 연산자가 있어야함. 그렇지 않으면 `onEach`만 호출하는 것으로는 효과가 없음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking {
    events()
        .onEach { event -> println("Event: $event") }
        .collect()
    println("Done")
}

실행결과
Event: 1
Event: 2
Event: 3
Done
```

- `luanchIn` 터미널(종단)연산자가 여기서 편리하게 사용될 수 있음.
- `collect`를 `luanchIn` 으로 대체함으로써 Flow의 수집을 별도의 코루틴에서 실행할 수 있음.
    - 이후 코드들은 즉시 계속해서 실행될 수 있음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking {
    events()
        .onEach { event -> println("Event: $event") }
        .launchIn(this)
    println(this)
}

실행결과
"coroutine#1":BlockingCoroutine{Active}@394e1a0f
Event: 1
Event: 2
Event: 3
```

- `luanchIn` 에서 필요로 하는 CoroutineScope 파라미터는 `CoroutineScope`를 특정해 `Flow`가 실행되면 어떤 코루틴이 수집할 지 결정함.
    - 위 예제에선 `CoroutineScope`는 `runBlocking` 코루틴 빌더로부터 와서, `Flow`가 실행되는 동안 `runBlocking` 스코프가 자식 코루틴이 완료될 때까지 기다리도록 함. 그 이후에 main 함수를 반환하는 것을 방지하여 예제가 종료되지 않도록 함.
- 실제 앱에서는 생명주기를 가지는 곳에서 Scope를 가져옴.
    - 생명주기가 종료되는 순간 해당 Scope는 취소되며, Flow의 수집은 중단됨.
        - ex) `viewModelScope`
    - `onEach { .. }.launchIn(scope)` 쌍은 `addEventListener` 같이 동작함.
        - 취소와 구조화된 동시성이 `removeEventListener` 역할을 해주므로, 대신 수행해줄 필요가 없음.
- `launchIn` 또한 전체 Scope를 `cancel` 하거나 `join` 하지 않고 해당 `Flow`를 수집하는 코루틴만을 `cancel`하기 위해 사용할 수 있는 `Job`을 반환함.

```kotlin
public fun <T> Flow<T>.launchIn(scope: CoroutineScope): Job = scope.launch {
    collect() 
}
```

- `launchIn` 내부에서는 설정한 스코프를 가지고 `launch`를 해주고 있으며, 수신객체(`Flow`)에 대하여 `colelct`를 해주고 있음.
    - `launch`를 해주므로, 다른 코루틴에서 수집하는 것임.
        - 이게 `Job`을 반환하는 이유.

[https://developer88.tistory.com/entry/Coroutine-정리노트-Part2-Flow-Channel](https://developer88.tistory.com/entry/Coroutine-%EC%A0%95%EB%A6%AC%EB%85%B8%ED%8A%B8-Part2-Flow-Channel)

### Flow 취소 체크

- flow 빌더는 추가적으로 방출된 각 값에 대한 취소 동작을 하기 위한 `ensureActive` 체크를 수행함.
- 이는 `flow { .. }` 에서 루프를 돌며 바쁘게 방출되는 것이 취소 가능하다는 것을 뜻함.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun foo(): Flow<Int> = flow {
    for (i in 1..5) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    foo().collect { value ->
        if (value == 3) cancel()
        println(value)
    }
}

실행결과
Emitting 1
1
Emitting 2
2
Emitting 3
3
Emitting 4
Exception in thread "main" kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled; job=BlockingCoroutine{Cancelled}@33f88ab
```

- 숫자를 3까지만 소모하고 4를 방출한 다음 예외가 발생함.
- 하지만, 다른 대부분의 Flow 연산자들은 성능상의 이유로 추가적인 취소 체크를 하지 않음.
    - ex) `IntRange.asFlow` 다음과 같은 바픈 루프를 작성하기 위해 사용하고 아무 곳에서 일시 중단 하지 않는 다면 취소를 위한 체크는 일어나지 않음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    (1..5).asFlow().collect { value ->
        if (value == 3) cancel()
        println(value)
    }
}

실행결과
1
2
3
4
5
Exception in thread "main" kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled; job=BlockingCoroutine{Cancelled}@1c3a4799
```

- 1부터 5까지 모든 숫자들이 수집되고, `runBlocking` 이 반환되기 전에만 취소가 감지됨.

```kotlin
일시중단지점이 있어야 취소 동작 체크를 수행함.
```

### 바쁜 Flow를 취소 가능하게 만들기

- 코루틴에 바쁜 루프가 존재한다면 명시적으로 취소 체크를 해야함.
    - `onEach { currentCoroutineContext().ensureActive() }` 를 추가할 수 있지만
    - `cancellable` 연산자가 해당 역할을 수행하기 위해 이미 정의되어 있음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    (1..5).asFlow().cancellable().collect { value ->
        if (value == 3) cancel()
        println(value)
    }
}

fun main() = runBlocking<Unit> {
    (1..5).asFlow()
        .onEach { currentCoroutineContext().ensureActive() }
        .collect { value ->
            if (value == 3) cancel()
            println(value)
        }
}

실행결과
1
2
3
Exception in thread "main" kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled; job=BlockingCoroutine{Cancelled}@1d29cf23
```

- 둘의 실행결과는 같음.

### ****Flow와 Reactive Stream****

- 리액티브 스트림이나 RxJava나 Project Reactor 같은 리액티브 프레임웍에 익숙한 사람들은 Flow를 설계 하는게 아주 익숙하다고함.
- 실제로, Flow의 설계는 리액티브 스트림과 그에 대한 다양한 구현체들에 영감을 받았음. 하지만, Flow의 주요 목표는 가능한 단순하게 디자인을 하는 것임.
    - 코틀린의 일시중단, 구조적인 동시성을 존중하는 것.
