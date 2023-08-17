### Transform 연산자

- Flow의 변환 연산자들 중에서 가장 일반적인 것은 `transform`임.
    - 이는 `map`, `filter`와 같은 간단한 변환을 모방하거나, 복잡한 변환을 구현함.
        - 즉, 중간 연산으로 작동하면서 스트림(방출된 데이터)의 각 항목을 변환할 수 있음.
- `transform` 연산자를 사용하면 임의의 횟수 만큼 `emit` 할 수 있음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

suspend fun performRequest(request: Int): String {
    delay(1000)
    return "response $request"
}

fun main() = runBlocking<Unit> {
    (1..3).asFlow()
        .transform { request ->
            emit("Making request $request")
            emit(performRequest(request))
        }
        .collect { response -> println(response) }
}

실행결과
Making request 1
response 1
Making request 2
response 2
Making request 3
response 3
```

- 위 예제는 비동기 요청을 하기 전에 문자열을 방출함.

```kotlin
https://jizard.tistory.com/477
메모리 사용량 관점에서 map이 사용량이 더 적음. map 내부적으로 transform을 가지고 있음.
```

### 크기 한정 연산자

- `take` 와 같은 크기 한정 중간 연산자들은 임계치에 도달했을 때, flow의 실행을 취소함.
    - 파라미터가 양수가 아닌경우 `IllegalArgumentException` 발생
- 코루틴의 취소는 언제나 예외를 `throw` 하여 수행됨.
    - `try ~ finally`같은 모든 리소스 관리를 위한 기능들은 취소에 정상작동함.
        - `Flow` 연산 중에 발생하는 예외는 일반적으로 플로우 연산자나 종료 연산자에서 처리됨.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun numbers(): Flow<Int> = flow {
    try {
        emit(1)
        emit(2)
        println("This line will not execute")
        emit(3)
    } finally {
        println("Finally in numbers")
    }
}

fun main() = runBlocking<Unit> {
    numbers().take(2).collect { println(it) }
}

실행결과
1
2
Finally in numbers
```

### Flow 터미널 연산자(종단 연산자)

- `Flow`의 터미널 연산자는 flow의 수집을 시작하는 일시 중단 함수임.
    - 다양한 `Collection`으로 변환을 수행하는 `toList`, `toSet`
    - 첫 값만 가져오는 `first`와 하나의 값만 방출되는 것을 확인하는 `single`
    - flow를 값으로 줄이는 `reduce`, `fold`
        - `reduce` 는 초깃값 없이 0으로 시작하지만, `fold`는 파라미터로 초깃값을 줄 수 있음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    val sum = (1..5).asFlow()
        .map { it * it }
        .reduce { a, b -> a + b }
    println(sum)
}

실행결과
55
```

### Flow는 순차적이다

- 여러 `Flow`들에서 작동하는 특수한 연산자를 사용하지 않는 한 개별 Flow의 `Collection` 은 순차적으로 동작함.
- `Collection` 은 터미널(종단) 연산자를 호출하는 코루틴에서 직접 동작함.
    - 기본적으로 어떠한 새로운 코루틴도 실행되지 않음.
- 방출된 값들은 중간 연산자들에 의해 업 스트림에서 다운 스트림으로 처리된 후 터미널 연산자에게 전달됨.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    (1..5).asFlow()
        .filter {
            println("Filter $it")
            it % 2 == 0
        }
        .map {
            println("Map $it")
            "string $it"
        }
        .collect {
            println("Collect $it")
        }
}

실행결과
Filter 1
Filter 2
Map 2
Collect string 2
Filter 3
Filter 4
Map 4
Collect string 4
Filter 5
```

### Flow Context

- `Flow`의 수집은 언제나 코루틴을 호출하는 Context상에서 일어남.
- `Flow`의 구체적인 구현과 상관없이 개발자가 지정한 Context에서 실행됨.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow {
    println("Started simple flow ${Thread.currentThread().name}")
    for (i in 1..3) {
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    withContext(coroutineContext) {
        simple().collect { value ->
            println("${Thread.currentThread().name} $value") // run in the specified context
        }
    }
}

실행결과
Started simple flow main @coroutine#1
main @coroutine#1 1
main @coroutine#1 2
main @coroutine#1 3
```

- Flow의 이러한 성질은 `컨텍스트 보존(context preservation)`이라 불림.

### ****withContext를 사용할 때 일반적으로 겪을 수 있는 함정****

- 일반적으로 `withContext`는 코루틴을 사용하는 코드의 Context를 변경하는데 사용됨.
    - 하지만, `Flow`는 `컨텍스트 보존` 특성을 준수해야해서 다른 컨텍스트에서 방출(`emit`)하는 것은 허용하지 않음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow {
    withContext(Dispatchers.Default) {
        for (i in 1..3) {
            emit(i) 
        }
    }
}

fun main() = runBlocking<Unit> {
    simple().collect { value -> println(value) }
}

실행결과
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
		Flow was collected in [CoroutineId(1), "coroutine#1":BlockingCoroutine{Active}@2b4a1161, BlockingEventLoop@338da300],
		but emission happened in [CoroutineId(1), "coroutine#1":DispatchedCoroutine{Active}@1178e2a5, Dispatchers.Default].
		Please refer to 'flow' documentation or use 'flowOn' instead
```
