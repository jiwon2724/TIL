# 코루틴(Coroutine) 기초

- 코루틴은 일시정지 연산을 위한 인스턴스임.
- 코루틴은 스레드와 비슷하지만, 특정한 스레드에 종속되어 실행되지 않음.
- 하나의 스레드에서 일시정지(suspend) 되었다가 다른 스레드에서 재개(resume)될 수 있음.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch { // 백그라운드에서 새 코루틴 시작
        delay(1000)
        println("World!")
    }
    println("Hello")
}
```

출력결과

```kotlin
Hello
World!

// 1초후에 World!가 출력됨
```

- `launch` :  Coroutine Builder이고, 독립적으로 동작하는 새로운 코루틴을 나머지 코드와 동시에 실행되도록 함.
    - 즉, `launch` 코드블록과 `println("Hello")` 를 동시에 실행함.
    - `launch` 블록으로 감싸면 비동기로 실행이 됨.
- `delay` : 일시중단 함수임. 이는 코루틴을 지정한 시간동안 일시중단함.
    - 코루틴이 실행중인 Thread는 블록하지않음.
    - `delay` 함수가 호출되면, 일시 중단되는 동안 다른 코루틴이 실행되도록 양보함.
    - 설정한 시간이 지나면, `delay` 함수는 코루틴을 다시 활성화하고 중단 지점부터 다시 재개됨.
    - 이는 non-blocking 방식으로 비동기적은 동작을 구현하는데 도움을 줌.
- `runBlocking` : Coroutine Builder이고, 블록내의 코드를 다 수행할때까지 현재 스레드를 블록시킴.
    - 위 예제에선 Main Thread를 블록시킴.

### 구조화된 동시성

- 코루틴은 새로운 코루틴의 수명을 제한하는 특정한 CoroutineScope 내에서만 실행될 수 있다는 원칙을 따름.
    - 이 원칙은 `구조화된 동시성` 임.
    - 부모 - 자식 계층관계를 형성함.
        - `runBlocking` : 부모
        - `launch` : 자식
- 위 예시에선 `runBlocking` 이 해당 스코프를 만들며, 1초를 기다린후 종료되는 이유임.
- `구조화된 동시성` 은 코루틴들이 손실되거나 누수를 일으키지 않도록 함.
    - 바깥 Scope(부모)는 자식 코루틴들이 모두 완료될 때까지 완료되지 못함.

```kotlin
GlobalScope는 구조화된 동시성의 원칙을 따르지 않음.
https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/
```

### 함수 추출해 리펙토링하기

- 일시중단 함수로 코루틴 내부에서 사용되는 로직을 추출할 수 있음.
    - `suspend` 키워드를 붙여 함수를 작성함.
    - 다른 일시 중단 함수를 사용하여 코루틴 실행을 일시중단 할 수 있음.
        - 다른 일시 중단 함수 ex : `delay` , 다른 `susepnd` 함수.
            - `delay` 도 일시중단함수임.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch { doWorld() }
    println("Hello")
}

suspend fun doWorld() {
    delay(1000)
    println("World!")
}
```

### Scope Builder

- `coroutineScope` 빌더를 사용하여 고유한 스코프를 선언할 수 있음.
    - 이는 자식 코루틴들의 실행이 모두 완료될 때까지 종료되지 않는 코루틴 스코프를 생성함.
    - `runBlocking` 과 차이점은 `delay` 함수 호출 시 Thread를 블록 시키는 방면 `coroutineScope` 는 다른 작업이 수행될 수 있도록 코루틴을 일시정지함.
        - 위 같은 이유로 `runBlocking` 은 일반 함수 `coroutineScope` 는 일시 중단 함수임.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    doWorld()
}

suspend fun doWorld() = coroutineScope{
    launch {
        delay(1000L)
        println("World!")
    }
    println("Hello")
}
```

### Scope Builder와 동시성

- `coroutineScope` 빌더는 일시 중단 함수 내부에서 복수의 동시작업을 수행하기 위해 사용될 수 있음.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    doWorld()
    println("Done")
}

suspend fun doWorld() = coroutineScope {
    launch {
        delay(2000L)
        println("World 2")
    }

    launch {
        delay(1000L)
        println("World1")
    }
    println("Hello")
}
```

- `coroutineScope` 는 두 개의 `launch` 가 모두 완료된 후 종료되며, doWorld 함수가 반환됨.

### Job 명시적으로 사용하기

- `launch` 코루틴 빌더는 실행된 코루틴을 처리하고 완료를 명시적으로 기다리도록 할 수 있음.
    - 이는 `Job` 객체를 반환함.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = launch { 
        delay(1000L)
        println("World!")
    }
    println("Hello")
    job.join() // 코루틴이 완료될 때까지 기다린다.
    println("Done")     
}
```

### 코루틴은 경량(light-weight)이다

- 코루틴은 실행 중인 스레드를 차단하지 않고, 일시중단을 지원하므로 단일 스레드에서 많은 코루틴을 실행 가능.
- block은 메모리를 차지하고 있는 반면에 일시중단은 메모리를 차지하지 않고 있으므로 메모리 절약 가능.
    - block : `Thread.sleep()`
    - 일시중단 : `suspend`

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    repeat(50_000) { // 많은 수의 코루틴을 실행한다.
        launch {
            delay(5000L)
            print(".")
        }
    }
}
```

위 코드를 Thread로 바꾼다면 많은 메모리를 사용하게 되고, 버전 환경에따라 OOM을 발생시키거나 스레드를 느리게 시작하게됨.
