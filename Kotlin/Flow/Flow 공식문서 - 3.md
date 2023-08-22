# Flow 공식문서 - 3

### flowOn 연산자

- `컨텍스트 보존` 특성을 준수해야해서 다른 컨텍스트에서 방출(`emit`)하는 것은 허용하지 않음.
- `컨텍스트 보존` 특성을 준수하지 않고, 다른 컨텍스트에서 방출하면 다음과 같은 예외가 나옴.

```kotlin
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
		Flow was collected in [CoroutineId(1), "coroutine#1":BlockingCoroutine{Active}@2b4a1161, BlockingEventLoop@338da300],
		but emission happened in [CoroutineId(1), "coroutine#1":DispatchedCoroutine{Active}@1178e2a5, Dispatchers.Default].
		Please refer to 'flow' documentation or use 'flowOn' instead
```

- `flowOn`함수는 `Flow`에서 값 방출을 위한 Context를 변경하는데 사용됨.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        Thread.sleep(100)
        log("Emitting $i")
        emit(i)
    }
}.flowOn(Dispatchers.Default)

fun main() = runBlocking {
    simple().collect { log("Collected $it")}
}

실행결과
[DefaultDispatcher-worker-1 @coroutine#2] Emitting 1
[main @coroutine#1] Collected 1
[DefaultDispatcher-worker-1 @coroutine#2] Emitting 2
[main @coroutine#1] Collected 2
[DefaultDispatcher-worker-1 @coroutine#2] Emitting 3
[main @coroutine#1] Collected 3
```

- `flowOn`연산자는 Context에서 `CoroutineDispatcher` 를 변경해야 할 때 업스트림 `Flow`를 위한 다른 코루틴을 생성함.

### Buffering

- 다른 코루틴속 `Flow`의 다른 부분들의 실행은 `Flow`를 수집(`collect`)하는데 걸리는 전체 시간의 관점에서 유용할 수 있음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlin.system.measureTimeMillis

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        println("emit 호출")
        emit(i)
    }
}

fun main() = runBlocking {
    val time = measureTimeMillis {
        simple().collect {
            delay(300)
            println(it)
        }
    }
    println("Collected in $time ms")
}

실행결과
emit 호출
1
emit 호출
2
emit 호출
3
Collected in 1242 ms
```

- 전체 수집 작업이 1200ms정도 걸림.
- `buffer` 연산자를 사용해 방출(`emit`) 코드와 수집(`collect`) 코드를 순차적으로 실행되도록 하는 대신 동시에 실행되도록 할 수 있음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlin.system.measureTimeMillis

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        println("emit 호출")
        emit(i)
    }
}

fun main() = runBlocking {
    val time = measureTimeMillis {
        simple()
            .buffer()
            .collect {
                delay(300)
                println(it)
            }
    }
    println("Collected in $time ms")
}

실행결과
emit 호출
emit 호출
emit 호출
1
2
3
Collected in 1046 ms
```

- 즉 버퍼는 업스트림과 다운스트림 사이의 동시성을 높여 생산과 소비의 속도 차이를 완화하는 역할임.

```kotlin
버퍼는 방출과 수집하는 코루틴을 분리함. 발행과 수집이 분리된다면, 수집이 완료되었을 때, 방출에 대한 delay없이 바로 방출이 가능함.
```
### Conflation

- Flow가 연산의 상태 갱신에 대한 일부 결과를 나타내는 경우 각 값을 처리할 필요가 없음.
    - 최신값만 처리하면 됨.
- 이러한 경우, 수집자(`collect`)가 너무 느리게 값들을 처리하는 경우 중간 발행 값들을 건너 뛸 수 있음.
    - `conflate` 연산자 사용

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlin.system.measureTimeMillis

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        println("emit 호출")
        emit(i)
    }
}

fun main() = runBlocking {
    val time = measureTimeMillis {
        simple()
            .conflate() 
            .collect { value ->
                delay(300) 
                println(value)
            }
    }
    println("Collected in $time ms")
}

실행결과
emit 호출
emit 호출
emit 호출
1
3
Collected in 741 ms
```

- 호출 순서가 위 예제인 버퍼를 사용했을 때와 비슷함.
- `conflate` 함수도 내부적으로는 `buffer` 를 사용하고있음.
    - 즉 버퍼에 값을 저장 후, 수집기(`collect`)쪽에서 버퍼에 저장된 값을 느리게 처리 시 중간 값들을 건너뜀.

```kotlin
public fun <T> Flow<T>.conflate(): Flow<T> = buffer(CONFLATED)
```

### 최신 값 처리하기

- 결합(Conflation)은 다운스트림, 업스트림 양쪽이 모두 느린 경우에 처리를 빠르게 하기 위해 사용할 수 있는 방법임.
- 결합은 방출된 값들을 삭제하여 처리를 빠르게 함.
- 다른 방법으론 느린 수집기(`collect`)의 실행을 취소하고, 새로운 값이 발행(`emit`)될 때마다 다시 시작하는 것임.
    - `collectLatest` 연산자 사용

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlin.system.measureTimeMillis

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        println("emit 호출")
        emit(i)
    }
}

fun main() = runBlocking {
    val time = measureTimeMillis {
        simple()
            .collectLatest { value -> // cancel & restart on the latest value
                println("Collecting $value")
                delay(300) // pretend we are processing it for 300 ms
                println("Done $value")
            }
    }
    println("Collected in $time ms")
}

실행결과
emit 호출
Collecting 1
emit 호출
Collecting 2
emit 호출
Collecting 3
Done 3
Collected in 673 ms
```

- `collectLatest` 의 블록에서 300ms의 시간이 걸리는 반면, 업스트림에선 새로운 값을 100ms 마다 방출(`emit`)됨.
- `collectLatest` 에 방출된 값은 출력되지만, 마지막 값에 대해서만 완료됨.
    - `println("Done $value")`
