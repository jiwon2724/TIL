# CoroutineContext와 Dispatcher

- 코루틴은 언제나 `CoroutineContext` 타입 값으로 표현되는 일부 Context 상에서 실행됨.
- `CoroutineContext`는 다양한 요소의 집합임 주요 요소는 다음과 같음.
    - `Job`
    - `Dispatchers`
    - `ExceptionHandler`
    - `CoroutineName`

```kotlin
CoroutineContext는 실행 환경을 설명하는 정보의 집합임. 어떻게 동작할지, 어디에서 실행될지
어떤 속성들을 가질지 등에 대한 정보를 포함함.
```

### Dispatchers와 Threads

- `CoroutineContext` 는 해당 코루틴의 실행에 사용되는 단일 스레드나, 복수의 스레드를 결정하는 `CoroutineDispatcher`가 포함됨.
- 이는 코루틴 실행에 사용될 스레드를 특정 스레드로 제한하거나, 스레드풀에 분배하거나, 제한 없이 실행 되도록함.
- 코루틴 빌더들은 `CoroutineContext` 파라미터를 선택적으로 받을 수 있음.
    - `CoroutineContext` 를 파라미터로 지정 안하면(default) 실행되는 `CoroutineScope` 으로 부터 Context를 상속받음.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    launch { println("main runBlocking : ${Thread.currentThread().name}") }
    launch(Dispatchers.Unconfined) { println("Unconfined : ${Thread.currentThread().name}") }
    launch(Dispatchers.Default) { println("Default : ${Thread.currentThread().name}") }
    launch(newSingleThreadContext("MyOwnThread")) { println("newSingleThreadContext : ${Thread.currentThread().name}") }
}

// 실행결과
Unconfined            : I'm working in thread main
Default               : I'm working in thread DefaultDispatcher-worker-1
newSingleThreadContext: I'm working in thread MyOwnThread
main runBlocking      : I'm working in thread main
```

- `launch` 에 파라미터가 정의되어 있지 않으므로 `runBlocking` 코루틴으로부터 Context를 상속받아 메인스레드에서 실행됨.
- `Dispatchers.Unconfined` : 특정 스레드에 국한되지 않는 디스패처임.
    - 호출 스레드에서 즉시 실행됨. → 위 예제에선 main 스레드에서 실행.
        - 첫 번째 일시 중단 포인트에서 중단되며, 재개 위치는 종료된 스레드에서 재개되지 않을 수 있음.
            - ex) main 스레드에서 일시중지 → 재개 → Default 디스패처 스레드에서 재개
                - 이 이후엔, 일시 중단 함수에 의해 결정된 스레드에서 코루틴을 재개함.
- `Dispatchers.Default` : 스코프내에서 다른 `Dispatcher` 사용이 명시적이지 않았을 때 사용됨.
    - 스레드들이 공유하는 Bacgkround Pool을 사용함
        - 이는 JVM, Native의 공유 스레드 풀에서 지원됨.
            - 사용되는 최대 스레드 수는 CPU의 코어수이고, 최소 2개임.
    - 백그라운드에서 CPU 집중적인 작업을 수행하기 위한 디스패처임.
        - 정렬, 파싱, 계산 등
- `Dispatchers.IO` : I/O, 네트워크 작업 등을 수행하는 최적화된 디스패처임.
    - CPU 작업에는 적합하지 않음.
- `newSingleThreadContext` : 코루틴이 실행되기 위한 새로운 단일 스레드를 생성함.
    - 전용 스레드는 매우 비싼 리소스임.
    - 앱에서 더 이상 필요하지 않을 때, `close` 함수를 사용해서 해제해야함.

### ****Coroutines와 Threads 디버깅 하기****

- 코루틴은 하나의 스레드에서 일시 중단한 다음 다른 스레드에서 재개될 수 있음.
- 싱글 스레드를 가진 Dispatcher에서도 특별한 도구가 없으면 Coroutine이 언제, 어디서 무엇을 하는지 알기 어려움.
- IDE에서 디버깅을 간단하게 할 수 있음.
    - https://kotlinlang.org/docs/debug-coroutines-with-idea.html#debug-coroutines

### 로깅을 통해 디버깅 하기

- 로그에 스레드의 이름을 넣어 출력할 수 있음.

```kotlin
import kotlinx.coroutines.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() = runBlocking<Unit> {
    val a = async {
        log("I'm computing a piece of the answer")
        6
    }
    val b = async {
        log("I'm computing another piece of the answer")
        7
    }
    log("The answer is ${a.await() * b.await()}")    
}

// 실행결과
[main @coroutine#2] I'm computing a piece of the answer
[main @coroutine#3] I'm computing another piece of the answer
[main @coroutine#1] The answer is 42
```

### Thread 전환 하기

- 단일 스레드를 만들어서 코루틴 빌더의 파라미터로 넣어서 코루틴이 실행되는 Thread를 전환할 수 있음.
- `use` 를 사용하여 `newSingleThreadContext` 생성된 스레드들을 사용하지 않을 때 해제함.

```kotlin
newSingleThreadContext("Ctx1").use { ctx1 ->
    newSingleThreadContext("Ctx2").use { ctx2 ->
        runBlocking(ctx1) {
            log("Started in ctx1")
            withContext(ctx2) {
                log("Working in ctx2")
            }
            log("Back to ctx1")
        }
    }
}
```
