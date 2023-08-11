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

### Context 내부의 Job

- 코루틴의 `Job` 은 `CoroutineContext` 의 구성요소임.
- `coroutineContext[Job]` 표현식으로 정보를 가져올 수 있음.
- `CoroutineScope` 의 `isActive` 는
    - `coroutineContext[Job]?.isActive == true` 의 축약임.

```kotlin
fun main() = runBlocking<Unit> {
    println("Job : ${coroutineContext[Job]}")
}

// 실행결과
Job : BlockingCoroutine{Active}@6d7b4f4c
```

### Coroutine의 자식들

- 코루틴이 다른 코루틴의 `CoroutineScope`에서 실행되면 `CoroutineContext` 를 상속받음.
    - 즉, 부모 코루틴의 `CoroutineContext` 를 상속.
- `CoroutineContext` 를 상속받은 새로운 코루틴의 `Job` 은 부모 코루틴 `Job` 의 자식이됨.
- 부모 코루틴이 취소되면, 자식 코루틴들 또한 취소됨.
- 부모 - 자식 관계는 두가지 방법으로 명시적으로 재정의 될 수 있음.
    1. 코루틴을 실행할 때 두 개의 다른 스코프에서 명시적으로 설정하는 경우
    2. 새로운 코루틴을 위해 다른 `Job` 이 Context로 전달되는 경우
        1. 부모 스코프의 `Job` 을 재정의
    
    두 경우 모두 실행된 코루틴은 실행된 스코프에 묶여있지 않고 독립적으로 동작.
    
    ```kotlin
    import kotlinx.coroutines.*
    
    fun main() = runBlocking<Unit> {
        val request = launch {
            launch(Job()) { // 부모 코루틴의 잡을 재정의
                println("job1: I run in my own Job and execute independently!")
                delay(1000)
                println("job1: I am not affected by cancellation of the request")
            }
    
            launch { // 자식 코루틴
                coroutineContext.is
                delay(100)
                println("job2: I am a child of the request coroutine")
                delay(1000)
                println("job2: I will not execute this line if my parent request is cancelled")
            }
    
            GlobalScope.launch { // 스코프를 명시적으로 설정
                println("job3: I run in GlobalScope and execute independently!")
                delay(1000)
                println("job3: I am not affected by cancellation of the request")
            }
        }
        delay(500)
        request.cancel() // cancel processing of the request
        println("main: Who has survived request cancellation?")
        delay(1000) // delay the main thread for a second to see what happens
    }
    ```
    
    ```kotlin
    job3: I run in GlobalScope and execute independently!
    job1: I run in my own Job and execute independently!
    job2: I am a child of the request coroutine
    main: Who has survived request cancellation?
    job1: I am not affected by cancellation of the request
    job3: I am not affected by cancellation of the request
    ```
    
    - 부모 코루틴이 취소 됐을 때, job1과 job3은 취소가 안된 걸 확인할 수 있음.
    
    ### 부모의 책임
    
    - 부모 코루틴은 언제나 자식들이 완료될 때 까지 기다림.
    - 부모는 모든 자식 코루틴들의 실행을 명시적으로 추적하지 못함
    - 모든 자식 코루틴들이 끝날 때까지 기다리기 위해 `Job.join` 을 사용할 필요가 없음.
    
    ```kotlin
    import kotlinx.coroutines.*
    
    fun main() = runBlocking<Unit> {
        val request = launch {
            repeat(3) { i -> // launch a few children jobs
                launch  {
                    delay((i + 1) * 200L) // variable delay 200ms, 400ms, 600ms
                    println("Coroutine $i is done")
                }
            }
            println("request: I'm done and I don't explicitly join my children that are still active")
        }
        request.join() // wait for completion of the request, including all its children
        println("Now processing of the request is complete")
    }
    ```
    ```kotlin
    실행순서를 출력하기 위해 join사용
    ```
    
    ### 디버깅을 위해 코루틴에 이름 짓기
    
    - `CoroutineContext` 요소인 `CoroutineName` 을 사용.
    - 이는 디버깅 모드가 켜져 있을 때, 코루틴을 실행하는 스레드 이름에 포함됨.
    - VM options에 `Dkotlinx.coroutines.debug` 추가
    
    ```kotlin
    import kotlinx.coroutines.*
    
    fun main(args: Array<String>) = runBlocking<Unit> {
        log("Started main coroutine")
        val v1 = async(CoroutineName("v1coroutine")) {
            delay(500)
            log("Computing v1")
            252
        }
        val v2 = async(CoroutineName("v2coroutine")) {
            delay(1000)
            log("Computing v2")
            6
        }
        log("The answer for v1 / v2 = ${v1.await() / v2.await()}")
    }
    
    fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
    ```
    
    ```kotlin
    [main @coroutine#1] Started main coroutine
    [main @v1coroutine#2] Computing v1
    [main @v2coroutine#3] Computing v2
    [main @coroutine#1] The answer for v1 / v2 = 42
    ```
    
    ### Context 요소들 결합하기
    
    - 종종 `CoroutineContext` 에 복수의 요소를 정의해야 할 수 있음.
    - `+` 연산자를 사용하여 복수의 요소를 정의할 수 있음.
        - 명시적으로 Dispatchers를 지정함과 동시에 이름을 지정한 코루틴 실행이 가능함.
    
    ```kotlin
    launch(Dispatchers.Default + CoroutineName("test")) {
        println("I'm working in thread ${Thread.currentThread().name}")
    }
    
    // I'm working in thread DefaultDispatcher-worker-1 @test#2
    ```
    
    ### CoroutineScope
    
    - 안드로이드 앱이 코루틴이 아닌 생명주기 객체를 가지고 있다고 가정해보자.
    - Activity 라이프 사이클에 맞춰서 소멸될 때, 메모리 누수를 방지하기 위해 코루틴들도 취소되어야 함.
    - Activity의 생명주기에 묶은 `CoroutineScope`의 인스턴스를 생성해 Coroutines의 생명주기를 관리할 수 있음.
        - 이는 팩토리 함수로 생성됨.
        - 즉, `CoroutineScope` 는 코루틴의 실행 범위(scope)를 정의하고, 이는 코루틴의 생명 주기와 연결되어 있으며, 특히 관련된 코루틴의 시작과 종료를 관리하는 데 중요한 역할을 함.
    
    ```kotlin
    class Activity {
        private val mainScope = MainScope()
        fun destroy() { mainScope.cancel() }
        fun doSomething() {
            repeat(10) { i ->
                mainScope.launch {
                    delay((i + 1) * 200L) // 
                    println("Coroutine $i is done")
                }
            }
        }
    }
    
    val activity = Activity()
    activity.doSomething()
    println("Launched coroutines")
    delay(500L)
    println("Destroying activity!")
    activity.destroy()
    delay(1000)
    ```
    
    ```kotlin
    Launched coroutines
    Coroutine 0 is done
    Coroutine 1 is done
    Destroying activity!
    ```
    
    - mainScope에서 2개의 코루틴들만 메세지를 호출함.
    - 그 이후 activity.destroy()가 실행되면서 mainScope가 취소됨.
    
    ### Thread-local 데이터
    
    - Thread-local 데이터란? → 특정 스레드에 한정된 데이터를 의미함.
        - 때때로 이를 코루틴으로 전달하는 기능이 유용할 때가 있음.
            - 코루틴은 특정 스레드에 국한 되지 않기 때문에, 이런 기능을 직접 구현하려면 번거로움.
            - ThreadLocal을 위한 `asContextElement` 확장함수를 사용할 수 있음.
                - 이는 주어진 ThreadLocal 의 값을 저장했다가 코루틴이 속한 컨텍스트를 변경할 때마다 복원함.
    
    ```kotlin
    import kotlinx.coroutines.*
    
    val threadLocal = ThreadLocal<String?>()
    
    fun main(args: Array<String>) = runBlocking<Unit> {
        threadLocal.set("main")
        println("Pre-main, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
        val job = launch(Dispatchers.Default + threadLocal.asContextElement(value = "launch")) {
            println("Launch start, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
            yield()
            println("After yield, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
        }
        job.join()
        println("Post-main, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
    }
    ```
    
    ```kotlin
    Pre-main, current thread: Thread[main,5,main], thread local value: 'main'
    Launch start, current thread: Thread[DefaultDispatcher-worker-1,5,main], thread local value: 'launch'
    After yield, current thread: Thread[DefaultDispatcher-worker-1,5,main], thread local value: 'launch'
    Post-main, current thread: Thread[main,5,main], thread local value: 'main'
    ```
    
    - 메인 스레드에서 `threadLocal.set("main")` 을 지정해줌.
        - 해당 스레드(특정 스레드)에 한정된 데이터임.
    - `Dispatchers.Default` 스레드에서 스레드 로컬의 값을 launch로 변경함.
    - 마지막 메인 스레드에서 다시 스레드 로컬을 호출해보면 “main”이 출력됨.
        - 코루틴이 속한 컨텍스트를 변경 했으므로, 값이 복원된 것.
