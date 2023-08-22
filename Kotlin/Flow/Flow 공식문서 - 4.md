# Flow 공식문서 - 4

### 여러 Flow 하나로 합치기

복수의 Flow를 합치는 다양한 방법이 있다.

- `zip`
- `combine`

### zip

- `Sequence.zip` 확장 함수처럼, Flow는 두 개의 Flow의 값을 결합하는 `zip` 연산자를 가지고 있음.

```kotlin
fun <T1, T2, R> Flow<T1>.zip(
    other: Flow<T2>, 
    transform: suspend (T1, T2) -> R
): Flow<R>
```

- 두 개의 `Flow`를 받아 새로운 `Flow` 를 리턴함.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    val nums = (1..3).asFlow()
    val str = flowOf("one", "two", "three")
    nums.zip(str) { a, b ->
        "$a -> $b"
    }.collect {
        println(it)
    }
}

실행결과
1 -> one
2 -> two
3 -> three
```

```kotlin
zip은 두 플로우증 하나의 플로우가 끝나면, 플로우가 취소됨.
```

### combine

- `Flow`가 가장 최신의 값이나 연산을 표현할 때, 해당 `Flow`의 가장 최신 값에 의존적인 연산의 수행을 필요로 하거나, 업 스트림이 새로운 값을 방출(`emit`)할 때 다시 연산할 수 있음.
    - 해당 연산을 수행하는 연산자의 집합이 `combine`임.
        - 즉, 여러 `Flow`의 동적인 변경사항들을 즉시 반영하여 결합 결과를 생성하고 방출함.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    val nums = (1..3).asFlow().onEach { delay(300) }
    val strs = flowOf("one", "two","three").onEach { delay(400) }
    val startTime = System.currentTimeMillis()

    nums.zip(strs) { a, b ->
        "$a -> $b"
    }.collect {
        println("$it at ${System.currentTimeMillis() - startTime} ms from start")
    }
}

실행결과
1 -> one at 422 ms from start
2 -> two at 828 ms from start
3 -> three at 1235 ms from start
```

```kotlin
fun <T> Flow<T>.onEach(action: suspend (T) -> Unit): Flow<T>
```

- `onEach` 는 다운 스트림으로 방출(`emit`)되기 전에 지정된 작업을 수행하는 `Flow`를 리턴함.
- `zip` 을 사용했을 때 400ms마다 출력되고, 동일한 결과가 생성됨.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    val nums = (1..3).asFlow().onEach { delay(300) }
    val strs = flowOf("one", "two","three").onEach { delay(400) }
    val startTime = System.currentTimeMillis()

    nums.combine(strs) { a, b ->
        "$a -> $b"
    }.collect {
        println("$it at ${System.currentTimeMillis() - startTime} ms from start")
    }
}

실행결과
1 -> one at 428 ms from start
2 -> one at 634 ms from start
2 -> two at 833 ms from start
3 -> two at 940 ms from start
3 -> three at 1239 ms from start
```

- combine을 사용한 경우 두 개의 Flow들이 각 방출에 따라 프린트 됨.
    - 두 플로우 중 하나라도 새 값을 방출하면 다른 플로우의 마지막 값을 사용하여 결합함.
    - 그러므로, nums `Flow`에서 1을 방출해도, 아직 strs `Flow`가 방출되지 않은 상태임.
        - nums `Flow`가 one을 방출했을 때, 결합되어 `1 -> one at 428 ms from start`이 출력됨.
        - 즉, 단일 `Flow`가 방출해도 결합 대상이 없다면 결합되지 않음.

```kotlin
Flow의 결합을 추적하는 경우엔 combine, 최종 결과만 필요한 경우는 zip을 사용해야 하는 것 같음.
https://velog.io/@sana/KotlinFlow-Flow-%EA%B2%B0%ED%95%A9-%EC%97%B0%EC%82%B0%EC%9E%90-zip-combine
```

### Flow를 Flatten하기

- Flow는 비동기적으로 수신된 값의 시퀀스를 나타냄.
- 다른 값들의 시퀀스에 대한 요청을 하기 매우 쉬움.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    (1..3).asFlow().map { requestFlow(it) }.collect {
        // it : Flow<String>
    }
}

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First")
    delay(500)
    emit("$i: Second")
}
```

- 이는 `Flow<Flow<String>>`이 되어버림.
- 이를 해결하기 위해 `Collection`과 `Sequence`는 `flatten`과 `flatMap` 연산자가 있음.
- Flow는 비동기 환경 때문에 flattening을 하기 위한 다른 방법이 필요함.

### flatMapConcat

- `flatMapConcat`과 `flattenConcat`은 `Flow`의 `Flow`에 대한 연결을 제공함.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    val startTime = System.currentTimeMillis() // remember the start time
    (1..3).asFlow().onEach { delay(100) } // emit a number every 100 ms
        .flatMapConcat { requestFlow(it) }
        .collect { value -> // collect and print
            println("$value at ${System.currentTimeMillis() - startTime} ms from start")
        }
}

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First")
    delay(500)
    emit("$i: Second")
}

실행결과
1: First at 120 ms from start
1: Second at 625 ms from start
2: First at 731 ms from start
2: Second at 1235 ms from start
3: First at 1340 ms from start
3: Second at 1846 ms from start
```

- `flatMapConcat` 은 순차적임.
    - 새로운 값을 수집하기 전에, 안쪽 `Flow`의 처리를 완료되기를 기다림.

### flatMapMerge

- 다른 flattening의 연산 방식은 수집 되는 값을 모두 동시적으로 수집한 후, 수집된 값들을 하나의 `Flow`로 만들어 빠르게 방출(`emit`)되도록함.
    - 이는 `flatMapMerge`, `flattenMerge`연산자에 의해 구현됨.
- 이 둘 모두 선택적으로 `concurrency` 파라미터를 받아 동시에 수직되는 `Flow`의 수를 제한할 수 있음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    val startTime = System.currentTimeMillis() // remember the start time
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms
        .flatMapMerge { requestFlow(it) }
        .collect { value -> // collect and print
            println("$value at ${System.currentTimeMillis() - startTime} ms from start")
        }
}

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First")
    delay(500)
    emit("$i: Second")
}

실행결과
1: First at 132 ms from start
2: First at 234 ms from start
3: First at 340 ms from start
1: Second at 637 ms from start
2: Second at 738 ms from start
3: Second at 849 ms from start
```

```kotlin
flatMapConcat은 순차적으로 플로우를 수집하는 반면, flatMapMerge는 여러 플로우를 동시에 수집함.
```

### ****flatMapLatest****

- `collectLatest` 연산자와 비슷하게, 새로운 Flow의 Collection이 방출되면 이전 Flow의 Collection처리가 취소됨. → 최신 값

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    val startTime = System.currentTimeMillis() // remember the start time
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms
        .flatMapLatest { requestFlow(it) }
        .collect { value -> // collect and print
            println("$value at ${System.currentTimeMillis() - startTime} ms from start")
        }
}

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First")
    delay(500)
    emit("$i: Second")
}

실행결과
1: First at 133 ms from start
2: First at 239 ms from start
3: First at 345 ms from start
3: Second at 853 ms from start
```

- 새로운 값이 수집되었을 때, `flatMapLatest` 블록 내부의 모든 코드를 취소함.
    - 즉, 새로운 값이 플로우에 방출될 때마다 현재 수집 중인 플로우를 취소하고 새로운 플로우로 전환함.
