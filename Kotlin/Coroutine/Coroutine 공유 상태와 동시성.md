# Coroutine 공유 상태와 동시성

- 코루틴은 `Dispatchers.Default` 와 같이 멀티 스레드를 관리하는 `Dispatcher` 에 의해 병렬적으로 실행될 수 있음.
- 병렬 실행 시 가장 중요한 문제는 `변경 가능한 공유 상태`의 동기화임.

### Coroutine을 여러개 실행했을 때의 문제점

```kotlin
import kotlinx.coroutines.*
import kotlin.system.measureTimeMillis

var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun { counter++ }
    }
}

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100
    val k = 1000
    val time = measureTimeMillis {
        coroutineScope {
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Counter = $counter")
}

실행결과 
Counter = 40101
```

- 복수의 스레드를 관리하는 `Dispatchers.Default` 를 사용해 공유 변수의 값을 증가시키는 동작임.
- 출력의 결과는 “Counter = 100000”을 출력할 가능성은 거의 없음.
    - 100개의 코루틴이 동기화 없이 여러 스레드에서 counter를 동시에 증가시킴.

### Volatile은 동시성 문제를 해결하지 못한다.

```kotlin
Volatile이란?
@Volatile을 붙이면 변수의 값이 메인 메모리에만 저장되며, 멀티 쓰레드 환경에서 메인 메모리의 값을 참조하므로 변수 값 불일치 문제를 해결할 수 있게된다.
다만 CPU캐시를 참조하는 것보다 메인메모리를 참조하는 것이 더 느리므로, 성능은 떨어질 수 밖에 없다.

https://www.charlezz.com/?p=45959
```

```kotlin
@Volatile
var counter = 0
```

- 변수를 `volatile` 로 만드는 것이 동시성 문제를 해결한다는 잘못된 인식이 있음.
- `volatile` 은 값을 증가시키는 것과 같은 동작에는 원자성을 제공하지 않음.

### Thread-safe한 데이터 구조

- 스레드와 코루틴에 모두 작동하는 일반적인 해결 방법은 공유 상태에 수행되어야하는 동작에 필수적인 동기화를 제공하는 스레드 안전한 데이터 구조를 사용하는 것임.
    - 스레드 안전한은 동기화된, 선형선, 원자성 이라고도 부름.

```kotlin
val counter = AtomicInteger()

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter.incrementAndGet()
        }
    }
}

실행결과 
Counter = 100000
```

- `AtomicInteger` 는 원자적인 동작을 제공함.

### 세밀하게 Thread 제한하기

- 스레드 제한
    - 단일 스레드로 제한된 공유 상태 문제에 대해 접근하기.
- 세밀하게 스레드를 제한하기 때문에 아주 느리게 동작함.

```kotlin
val counterContext = newSingleThreadContext("CounterContext")
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            withContext(counterContext) {
                counter++
            }
        }
    }
}
```

### 굵게 Thread 제한하기

- 큰 부분의 상태를 갱신하는 비즈니스 로직은 단일 스레드로 제한됨.
- 더 빠르게 실행됨.

```kotlin
val counterContext = newSingleThreadContext("CounterContext")
var counter = 0

fun main() = runBlocking {
    withContext(counterContext) {
        massiveRun { counter++ }
    }
}
```

### 상호 배제

```kotlin
상호 배제란?
여러 개의 스레드나 프로세스가 공유 자원에 동시에 접근하는 것을 방지하는 방법을 의미
```

- 문제에 대한 상호 배제 솔루션은 모든 공유 상태에 대한 변경을 절대로 동시에 실행되지 않는 영역으로 만들어 보호하는 것.
- 코루틴에서는 `Mutex`를 사용해서 영역을 구분하기 위한 `lock`과 `unlock`함수를 가짐.
    - `lock` 은 일시중단 함수임. 스레드를 블록하지 않음.
- `withLock` 함수는 다음 패턴을 나타내는 확장함수임.

```kotlin
withLock
mutex.lock()
try { ... } 
finally { mutex.unlock() }
```

```kotlin
val mutex = Mutex()
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            mutex.withLock {
                counter++
            }
        }
    }
    println("Counter = $counter")
}
```

- lock을 거는 것은 세밀하기 때문에 비용이 든다.
    - 이는 주기적으로 공유 상태를 수정해야 하지만, 상태를 제한시킬 수 있는 스레드가 없는 일부 상황에서 좋은 선택임.

### Actors

- `actor`는 코루틴의 조합으로 만들어진 엔티티임.
- `actor` 는 상태를 캡슐화하고 해당 상태를 동시에 여러 스레드에서 접근하는 것을 방지하는 방법임.
- `actor`는 다음과 같은 특징을 가짐.
    1. **메시지 패싱**: 액터는 메시지를 통해 상호작용함. 액터에게 작업을 지시하려면 메시지를 보내야 함.
    2. **상태 캡슐화**: 액터 내부의 상태는 액터 외부에서 직접 접근할 수 없음. 이는 동시성 문제를 방지하는 데 중요함.
    3. **순차적 처리**: 액터는 자신의 메일박스에 있는 메시지를 순차적으로 하나씩 처리함.

```kotlin
sealed class CounterMsg {
    object IncCounter : CounterMsg()
    class GetCounter(val response: CompletableDeferred<Int>) : CounterMsg()
}

fun main() = runBlocking<Unit> {
    val counter = counterActor()
    withContext(Dispatchers.Default) {
        massiveRun {
            counter.send(CounterMsg.IncCounter)
        }
    }
    val response = CompletableDeferred<Int>()
    counter.send(CounterMsg.GetCounter(response))
    println("Counter = ${response.await()}")
    counter.close()
}

fun CoroutineScope.counterActor() = actor<CounterMsg> {
    var counter = 0 // actor state
    for (msg in channel) { // iterate over incoming messages
        when (msg) {
            is CounterMsg.IncCounter -> counter++
            is CounterMsg.GetCounter -> msg.response.complete(counter)
        }
    }
}

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100
    val k = 1000
    val time = measureTimeMillis {
        coroutineScope {
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
}
```

- `actor`는 로드시 lock보다 효울적임.
    - 항상 해야 할 작업이 있으며, 다른 Context로 전환이 필요없음.
