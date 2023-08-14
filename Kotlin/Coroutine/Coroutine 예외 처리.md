# Coroutine 예외 처리

### Exception 전파

- 코루틴 빌더는 자동으로 예외를 전파하거나 사용자에게 예외를 노출함.
    - 자동으로 예외가 전파되는 코루틴 빌더 : `launch` , `actor`
    - 사용자에게 예외를 노출하는 코루틴 빌더 : `async` , `produce`
- 다른 코루틴의 자식이 아닌 root 코루틴을 코루틴 빌더로 만들 때
    - `launch` , `actor` 는 `uncaughtExceptionHandler`와 비슷하게 잡히지 않은 예외로 다룸.
    - `async` , `produce` 는 `await` 이나 `receive` 를 통해 사용자가 마지막 예외를 소비하는지에 의존.

```kotlin
import kotlinx.coroutines.*

@OptIn(DelicateCoroutinesApi::class)
fun main() = runBlocking {
		// 이 부분에서 GlobalScope를 빼면 예외가 전파되어 print를 호출하고 종료됨.
    val job = GlobalScope.launch { 
        println("Throwing exception from launch")
        throw IndexOutOfBoundsException()
    }
    job.join()
    println("joined failed job")
    val deferred = GlobalScope.async {
        println("Throwing exception from async")
        throw ArithmeticException()
    }
    try {
        deferred.await()
        println("Unreached")
    } catch (e: ArithmeticException) {
        println("Caught ArithmeticException")
    }
}

실행결과
Throwing exception from launch
Exception in thread "DefaultDispatcher-worker-1" java.lang.IndexOutOfBoundsException
	at MainKt$main$1$job$1.invokeSuspend(Main.kt:7)
	at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
	at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:108)
	at kotlinx.coroutines.scheduling.CoroutineScheduler.runSafely(CoroutineScheduler.kt:584)
	at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.executeTask(CoroutineScheduler.kt:793)
	at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.runWorker(CoroutineScheduler.kt:697)
	at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.run(CoroutineScheduler.kt:684)
	Suppressed: kotlinx.coroutines.internal.DiagnosticCoroutineContextException: [StandaloneCoroutine{Cancelling}@5428648a, Dispatchers.Default]
joined failed job
Throwing exception from async
Caught ArithmeticException
```

- `launch` 는 잡히지 않은 예외로 다룸.
    - 예외를 처리하는 코드가 없어 실제로 처리되지 않고, 잡히지 않은 예외가 발생하면 대부분의 환경은 프로그램이 강제 종료됨.
- `runBlocking` 코루틴 안에서 `GlobalScope.launch`를 사용함.
    - 기본적으로는 부모 코루틴에 의해 예외가 처리되지만 위 예제는 조금 다름.
    - 코루틴 안에서 스코프를 명시적으로 사용했음.
        - 즉, 계층관계가 재정의 됨. → 다른 코루틴과 독립적으로 실행
            - [코루틴의 자식들](https://www.notion.so/f0ae549691be4da8bebcab73ce8d154e?pvs=21)

### CoroutineExceptionHandler 사용해 전파된 예외 처리하기

- 잡히지 않은 예외를 콘솔에 출력하도록 기본 동작을 커스텀 할 수 있음.
- root 코루틴의 `CoroutineContext`인 `CoroutineExceptionHandler` 이 있음.
    - 이는 root 코루틴과 모든 자식 코루틴들에 대해 커스텀한 예외가 필요한 경우 `catch` 블록으로 사용됨.
    - 예외를 복구하지 못함.
        - 코루틴은 Handler가 호출되었을 때, 이미 해당 예외처리를 완료함.
    - 오류를 로깅하거나, 에러 메세지를 보여주거나 애플리케이션을 종료하거나 다시 시작하기 위해 사용됨.
    - 잡히지 않은 예외에 대해서만 실행됨.
        - 모든 자식 코루틴들(다른 Job의 Context로 만들어진 코루틴 포함)은 예외처리를 부모 코루틴에 위임함.
            - 그 부모 또한 부모에게 위임해서 루트 코루틴까지 올라감.
            - 따라서 그들의 `CoroutineContext`에 추가된 `CoroutineExceptionHandler`는 사용되지 않음.
    
    ```kotlin
    async 코루틴 빌더는 모든 예외를 잡아 Deferred 객체에 나타내므로
    CoroutineExceptionHandler가 아무런 효과가 없음.
    ```
    
    ```kotlin
    import kotlinx.coroutines.*
    
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler got $exception")
    }
    
    fun main() = runBlocking<Unit> {
        val job = GlobalScope.launch(handler) { // root coroutine, running in GlobalScope
            throw AssertionError()
        }
        val deferred = GlobalScope.async(handler) { // also root, but async instead of launch
            throw ArithmeticException() // Nothing will be printed, relying on user to call deferred.await()
        }
        joinAll(job, deferred)
    }
    
    실행결과
    CoroutineExceptionHandler got java.lang.AssertionError
    ```
    
    [https://myungpyo.medium.com/코루틴-공식-가이드-자세히-읽기-part-6-dd7796150ff3](https://myungpyo.medium.com/%EC%BD%94%EB%A3%A8%ED%8B%B4-%EA%B3%B5%EC%8B%9D-%EA%B0%80%EC%9D%B4%EB%93%9C-%EC%9E%90%EC%84%B8%ED%9E%88-%EC%9D%BD%EA%B8%B0-part-6-dd7796150ff3)
    
    - 궁금한 부분 코멘트 남김.

```kotlin
답변:
import kotlinx.coroutines.*

val handler = CoroutineExceptionHandler { _, exception ->
    println("CoroutineExceptionHandler got $exception")
}
val testHandler = CoroutineExceptionHandler { _, exception ->
    println("test got $exception")
}
fun main(): Unit = runBlocking {
    launch(handler) {
        launch(Job() + testHandler) {
            throw AssertionError()
        }
    }
}
```
- 위 코드에서 내부 launch는 Job을 새로 생성해서 전달하고 있음.
- 이는 runBlocking으로 생성된 루트 코루틴으로 부터 코루틴 계층에 속하지 않는 코루틴을 생성함.
  - 즉, 내부에 있는 `launch(Job() + testHandler)`이 루트 코루틴이 됨.
- `launch(Job() + testHandler)`이 루트 코루틴이므로, 예외는 handler가 아닌 testHandler에서 잡히는게 맞음.
### ****Cancellation과 Exceptions****

- 취소는 예외와 밀접하게 연관되어 있음.
- 코루틴은 내부적으로 취소를 위해 `CancellationException`을 사용함.
    - 이는 모든 Handler에서 무시됨.
    - `catch` 를 사용하여 추가적인 디버깅 정보를 위해서만 사용해야함.
- 코루틴이 Job.cancel()을 사용해 취소될 경우 종료되지만, 부모 코루틴의 실행을 취소하진 않음.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    val job = launch {
        val child = launch {
            try {
                delay(Long.MAX_VALUE)
            } finally {
                println("Child is cancelled")
            }
        }
        yield()
        println("Cancelling child")
        child.cancel()
        child.join()
        yield()
        println("Parent is not cancelled")
    }
    job.join()
}

실행결과
Cancelling child
Child is cancelled
Parent is not cancelled
```

- 코루틴은 CancellationException를 제외한 다른 예외들을 만난다면 부모 코루틴까지 취소됨.
    - 부모 코루틴이 취소되면서 자식 코루틴까지 예외가 전파됨.
- 구조화된 동시성을 위해 안정적인 코루틴 계층구조를 제공하기 위해 위 동작은 재정의 할 수 없음.
- `CoroutineExceptionHandler`의 구현식은 자식 코루틴들을 위해 사용되지 않음

### Exceptions 합치기

- 부모 코루틴 복수의 자식들이 예외와 함께 실행에 실패한다면 일반적인 규칙은 첫 번째 예외가 이김.
    - 즉, 첫 번째 예외만 처리됨.
- 첫 예외 이후 생긴 모두 추가적인 예외들은 첫번째 예외에 suppressed로 붙여짐.

```kotlin
import kotlinx.coroutines.*
import java.io.IOException

fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler got $exception with suppressed ${exception.suppressed.contentToString()}")
    }

    val job = launch(Job() + handler) {
        launch {
            try {
                delay(Long.MAX_VALUE)
            } finally {
                throw ArithmeticException()
            }
        }
        launch {
            delay(100)
            throw IOException()
        }
        delay(Long.MAX_VALUE)
    }
    job.join()
}

실행결과 
CoroutineExceptionHandler got java.io.IOException with suppressed [java.lang.ArithmeticException]
```

- suppressed에 `ArithmeticException` 예외가 출력됨..

### 취소 예외는 투명하고 기본적으로 감싸진다.

```kotlin
import kotlinx.coroutines.*
import java.io.IOException

fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler got $exception")
    }

    val job = GlobalScope.launch(handler) {
        val inner = launch { // all this stack of coroutines will get cancelled
            launch {
                launch {
                    throw IOException() // the original exception
                }
            }
        }
        try {
            inner.join()
        } catch (e: CancellationException) {
            println("Rethrowing CancellationException with original cause")
            throw e // cancellation exception is rethrown, yet the original IOException gets to the handler
        }
    }
    job.join()
}

실행결과
Rethrowing CancellationException with original cause
CoroutineExceptionHandler got java.io.IOException
```

- `IOException` 이 발생하면 가장 안쪽의 코루틴부터 바깥쪽의 코루틴으로 예외가 전파된다.
    - 예외가 전파되면서 각 코루틴은 `CancellationException` 으로 취소됨.
- `catch` 코드블록에서 예외를 리스로잉 해줌.
    - 이 때 `IOException` 이 `CancellationException` 의 원인(cause)로 설정되어 있음.
- `CancellationException` 예외가 다시 던져질 때, 원래의 예외도 함께 전파됨.

### ****Supervision****

- 취소는 코루틴의 전체 계층을 통해 전파되는 양방향 관계를 가짐.
- `Supervision` 은 예외 전파 동작을 변경하는 도구임.
    - ex) UI 구성요소에서 자식의 작업이 실패 되더라도, 모든 UI 구성요소를 취소할 필요는 없음.

### ****Supervision job****

- `SupervisorJob` 은 위 목적을 위해 사용된다.
- 이는 자식 코루틴의 실패가 다른 코루틴들에게 전파되지 않음.
- 취소가 아래 방향으로 전파되는 것만 제외하면 일반적인 `Job` 과 비슷함.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val supervisor = SupervisorJob()
    with(CoroutineScope(coroutineContext + supervisor)) {
        // launch the first child -- its exception is ignored for this example (don't do this in practice!)
        val firstChild = launch(CoroutineExceptionHandler { _, _ ->  }) {
            println("The first child is failing")
            throw AssertionError("The first child is cancelled")
        }
        // launch the second child
        val secondChild = launch {
            firstChild.join()
            // Cancellation of the first child is not propagated to the second child
            println("The first child is cancelled: ${firstChild.isCancelled}, but the second one is still active")
            try {
                delay(Long.MAX_VALUE)
            } finally {
                // But cancellation of the supervisor is propagated
                println("The second child is cancelled because the supervisor was cancelled")
            }
        }
        // wait until the first child fails & completes
        firstChild.join()
        println("Cancelling the supervisor")
        supervisor.cancel()
        secondChild.join()
    }
}

실행결과
The first child is failing
The first child is cancelled: true, but the second one is still active
Cancelling the supervisor
The second child is cancelled because the supervisor was cancelled
```

- firstChild 코루틴이 예외를 발생해도, 다른 코루틴엔 영향이 없음.

### ****Supervision Scope****

- 특정 범위에 대한 동시성을 적용하기 위해 `coroutineScope` 대신 `supervisorScope` 를 사용할 수 있음.
- 이는 취소를 한 방향으로만 전파하며, 그 자신이 실패했을 때만 자식 코루틴들을 취소함.

```kotlin
fun main() = runBlocking {
    try {
        supervisorScope {
            val child = launch {
                try {
                    println("The child is sleeping")
                    delay(Long.MAX_VALUE)
                } finally {
                    println("The child is cancelled")
                }
            }
            // Give our child a chance to execute and print using yield
            yield()
            println("Throwing an exception from the scope")
            throw AssertionError()
        }
    } catch(e: AssertionError) {
        println("Caught an assertion error")
    }
}

실행결과 
The child is sleeping
Throwing an exception from the scope
The child is cancelled
Caught an assertion error
```

### ****Supervise가 사용된 Coroutine에서의 예외****

- `Job` 과 `SupervisorJob` 의 또다른 중요한 차이는 예외처리임.
- 모든 자식은 자신의 예외를 예외 처리 메커니즘에 따라 직접 처리해야함.
    - 다른점은 자식의 실패가 부모에게 전파되지 않는점.
- `supervisorScope` 내부에서 직접 실행된 코루틴은 루트 코루틴과 비슷하게 스코프 내부에 `CoroutineExceptionHandler`를 쓰는 것을 뜻함.

```kotlin
import kotlinx.coroutines.*

val handler = CoroutineExceptionHandler { _, exception ->
    println("CoroutineExceptionHandler got $exception")
}

fun main() = runBlocking {
    supervisorScope {
        val child = launch(handler) {
            println("The child throws an exception")
            throw AssertionError()
        }
        println("The scope is completing")
    }
    println("The scope is completed")
}

실행결과
The scope is completing
The child throws an exception
CoroutineExceptionHandler got java.lang.AssertionError
The scope is completed
```
