### Flow 예외 처리

`Flow` 수집(`collect`)은 방출(`emit`)하는 곳이나 연산자 안의 코드가 예외를 발생시키는 경우 예외와 함께 완료될 수 있음.

### 수집기에서의 try와 catch

- 수집기(`collect`)는 예외를 처리하기 위해 `try/catch` 블록을 사용할 수 있음.
- 터미널(종단)연산자 안에서 예외를 잡으며, 그 뒤로 더이상 값이 방출(`emit`)되지 않음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value ->
            println(value)
            check(value <= 1) { "Collected $value" }
        }
    } catch (e: Throwable) {
        println("Caught $e")
    }
}

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i) // emit next value
    }
}

실행결과
Emitting 1
1
Emitting 2
2
Caught java.lang.IllegalStateException: Collected 2
```

### 모든 것이 잡힌다

- 방출하는 곳이나 중간 연산자, 터미널 연산자 안에서 예외를 모두 잡아냄.
- 방출된 값에 대한 연산의 코드도 예외를 발생시킴.

```kotlin
fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } catch (e: Throwable) {
        println("Caught $e")
    }
}

fun simple(): Flow<String> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}.map { value ->
    println("Map $value")
    check(value <= 1) { "Crashed on $value" }
    "string $value"
}

실행결과
Emitting 1
Map 1
string 1
Emitting 2
Map 2
Caught java.lang.IllegalStateException: Crashed on 2
```

### 예외 투명성(Exception ****Transparency****)

- `Flow`는 예외에 투명해야 함
- `try/catch` 블록 내부에서 `flow { .. }` 빌더의 값을 방출하는 것은 예외 투명성을 위반하는 것임.
- 예외를 발생 시키는 수집기는 언제나 `try/catch`를 사용해 예외를 잡아낼 수 있음.
- 방출기(방출하는 곳)는 `catch` 연산자를 사용해 예외 투명성을 유지시키고 예외 처리를 캡슐화 할 수 있음.
    - `catch` 연산자의 코드 블록은 예외를 분석하고, 잡은 예외에 따라 다른 방식으로 대응할 수 있음.

```kotlin
fun main() = runBlocking<Unit> {
    simple()
        .catch { e -> emit("Caught $e") } // emit on exception
        .collect { value -> println(value) }
}

실행결과
Emitting 1
string 1
Emitting 2
Caught java.lang.IllegalStateException: Crashed on 2
```

- 예외는 `throw` 를 통해서 다시 re-throw 될 수 있음.
- `catch` 의 코드 블록에서 `emit` 을 사용해 예외를 방출로 바꿀 수 있음.
- 예외는 무시되거나, 로깅되거나, 다른 코드로 처리될 수 있음.

### 투명한 catch

- 예외 투명도를 존중하는 `catch` 중간 연산자는 업스트림 예외만을 잡아냄.
    - `catch` 윗 부분의 연산자들에서의 예외만 잡아냄.
- `collect { .. }` 내부의 블록이 예외를 발생시키면 예외를 잡아내지 않음.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    simple()
        .catch { e -> println("Caught $e") } 
        .collect { value ->
            check(value <= 1) { "Collected $value" }
            println(value)
        }
}

실행결과
Emitting 1
1
Emitting 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
	at MainKt$main$1$2.emit(Main.kt:15)
  ...
```

### 선언적으로 잡기

- `collect` 연산자의 코드블록의 로직을 `onEach` 내부로 이동하고 `catch` 를 그 이후에 위치 시킴
    - `catch` 연산자의 선언적인 성질을 모든 예외들을 처리하기 위한 욕구와 결합이 가능함.
- `collect()` 를 파라미터 없이 사용함으로 `Flow`의 수집을 발생시킬 수 있음.

```kotlin
fun main() = runBlocking<Unit> {
    simple()
        .onEach { value ->
            check(value <= 1) { "Collected $value" }
            println(value)
        }
        .catch { e -> println("Caught $e") }
        .collect()
}

실행결과
Emitting 1
1
Emitting 2
Caught java.lang.IllegalStateException: Collected 2
```

- 명시적으로 `try/catch` 블록을 사용하지 않고도 모든 예외들을 잡아낼 수 있다.

### Flow 수집 완료 처리하기

- Flow의 수집이 완료(정상처리 or 예외발생)되면, 완료에 따른 동작을 실행이 가능함.

### ****명령적인 finally 블록****

- `try/catch` 에 더해 수집기(`collect`)는 동작이 완료됨에 따라 동작을 실행하는 `finally` 를 사용할 수 있음.

```kotlin
fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } finally {
        println("Done")
    }
}

실행결과
1
2
3
Done
```
