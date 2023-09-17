# CHAPTER 10 함수 추가사항 알아보기

### 순수함수와 일급 객체 함수

- 순수함수(Pure Function)
    - 동일한 입력에 대해 항상 동일한 출력을 반환함.
        - 오로지 입력 값에만 의존함.
        - 외부 상태나 데이터에 의존X
    - 함수 실행 중 외부의 상태를 변경하지 않음.
    - 함수 내부에서 어떤 부수 효과도 발생시키지 않음.
- 일급 객체 함수(First-Class Function)
    - 다음과 같은 특징을 만족함.
        - 변수에 할당될 수 있음.
        - 함수의 인자로 전달될 수 있음.
        - 함수의 결과로 반환될 수 있음.
        - 자료 구조에 저장될 수 있음.
    - 이러한 특징은 고차 함수(Higher-Order Function)를 쉽게 구현할 수 있음.

### 부수효과(Side Effect)

- 함수가 실행되는 과정에서 외부 데이터를 사용 및 수정하거나 외부의 다른 기능을 사용하는 것.
    - 함수가 전역변수를 사용하거나 수정하는 경우
    - I/O 작업
    - 객체의 상태를 변경
    - 등

```kotlin
// 순수 함수
fun add(a: Int, b: Int): Int = a + b // 동일한 입력에 동일한 출력 반환

// 순수 함수가 아님
var counter = 0
fun increment(): Int {
    counter += 1 // side effect
    return counter
}
```

### 일급 객체 함수 처리

```kotlin
fun main() {
    println(add1(10, 20))
    println(add2(1, 2))
    println(highFunc({ x: Int, y: Int -> x * y }, 100, 100))
    val rf = returnFunc()
    println(rf(3, 5))

// 일급 함수를 자료구조에 저장
    val map = mutableMapOf<String, (Int, Int) -> Int>()
    map["add"] = ::add
    map["mul"] = ::mul

    val operator = "*"
    when(operator) {
        "+" -> println(map["add"]?.invoke(10, 20))
        "*" -> println(map["mul"]?.invoke(10, 20)) // 200
        else -> println("없음")
    }
}

val add1 = fun (x: Int, y: Int): Int = x + y // 익명 함수 변수에 할당
val add2 = { x: Int, y: Int -> x + y } // 람다를 변수에 할당
fun highFunc(sum: (Int, Int) -> Int, a: Int, b: Int): Int = sum(a, b) // 함수의 인자로 전달
fun returnFunc(): (Int, Int) -> Int = { x: Int, y: Int -> x + y } // 반환 타입을 람다로 반환

fun add(a: Int, b: Int): Int = a + b
fun mul(a: Int, b: Int): Int = a * b

실행결과
30
3
10000
8
200
```

### 커링(Currying Function)함수 알아보기

- 여러 개의 인수를 받는 함수를 하나의 인수만을 받는 여러개의 함수로 분해하는 패턴임.
- 함수형 프로그래밍에서 일반적으로 사용되는 기법 중 하나임.

```kotlin
// fun add(x: Int, y: Int): Int = x + y add 함수를 커링함수로 바꿈.

fun main() {
    val add2 = addCurried(2)
    println(add2)
    val result = add2(3)
    println(result)
}

fun addCurried(x: Int): (Int) -> Int {
    return fun(y: Int): Int {
        return x + y
    }
}

실행결과
(kotlin.Int) -> kotlin.Int
5
```

### 메서드 체인처리

- 연속 호출을 위해 객체를 반환해야함.

```kotlin
class Person(var name: String, var age: Int) {
    fun introduce() { println("Hello, I'm $name and I'm $age years old.") }
    
    fun changeAge(newAge: Int): Person {
        this.age = newAge
        return this
    }
}

// 확장 함수로도 정의 가능
fun Person.changeName(newName: String): Person {
    this.name = newName
    return this
}

fun main() {
    val john = Person("John", 25)
        .changeName("Johnny")
        .changeAge(26)
        .introduce()  // 출력: Hello, I'm Johnny and I'm 26 years old.
}
```

### 고차 함수

- 함수를 객체로 생각해서 인자로 전달되거나 반환값으로 처리되는 함수.
    - 함수의 매개변수에 함수 자료형을 정의하고 함수를 호출할 때 인자로 다른 함수를 받음.
    - 함수 내부에서 반환값으로 다른 함수를 반환함.
    - 인자와 반환값으로 함수, 익명함수, 람다로 처리할 수 있음.
        - 함수일 경우엔 함수 이름이 아닌 함수 참조로 처리해야함.

```kotlin
typealias IntOperation = (Int, Int) -> Int

fun highFunction(vararg x: Int, op: IntOperation): Int = x.toList().reduce(op) // 함수를 매개변수로 받음.
fun add(x: Int, y: Int): Int = x + y
fun returnFunction(): IntOperation = { x, y -> x + y } // 함수 반환

fun main() {
    println(highFunction(1, 2, 3, 4, op = { x: Int, y: Int -> x + y })) // 람다 전달
    println(highFunction(1, 2, 3, 4, 5, op = ::add)) // 함수 참조 전달
    println(returnFunction()(10, 20))
}

실행결과
10
15
30
```

- `typealias` 는 Kotlin에서 타입에 대한 다른 이름(alias)을 제공하는 방법임.
- 주로 긴 제네릭 타입 표현식의 단순화나 코드의 가독성을 높이기 위해 사용됨.
- 단순히 새로운 타입을 생성하는 것이 아니라, 기존 타입에 대한 다른 이름을 제공하는 것.

### 합성 함수

- 두 함수를 하나의 함수로 연결한 것.
- 합성 함수를 구성하려면 함수의 매개변수와 자료형이 일치해야함.
    - 두 함수의 매개 변수와 반환 자료형이 같아야 함.
- 두 함수를 결합한 함수도 두 함수의 매개변수와 자료형이 같아야 함.
- 내부 처리는 전달되는 함수를 실행하고 그 결과를 받은 함수를 실행하는 순서로 처리.

```kotlin
fun addFive(x: Int): Int = x + 5
fun double(x: Int): Int = x * 2
fun addFiveThenDouble(x: Int): Int = double(addFive(x))

fun main() {
    println(addFiveThenDouble(10)) // 15
}
```

### 스코프 함수(범위 지정 함수)

- 특정 객체에 대한 작업을 블록 안에 넣어 실행할 수 있도록 하는 함수.
- 블록은 특정 객체에 대해 수행 할 작업의 범위가 되어서 범위 지정 함수라고도 부름.
- `apply`, `run` ,`with`, `let`, `also`가 있음.
    - apply, also는 수신객체 자체를 반환함.
    - run, with, let은 블록의 마지막 라인을 반환함.

### apply

```kotlin
// T는 apply의 수신객체.
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract { callsInPlace(block, InvocationKind.EXACTLY_ONCE)}
    block()
    return this
}
```

- apply는 수신객체 내부 프로퍼티를 변경한 다음 수신객체 자체를 반환하기 위해 사용되는 함수임.
- 객체 생성 시 다양한 프로퍼티를 설정해야 하는 경우 사용됨.
- block의 람다식은 apply의 수신객체 T를 지정하므로, 람다식 내부(코드블록)에서 수신객체에 대한 명시를 하지 않고 함수를 호출할 수 있음.

```kotlin
data class Person(
    var name: String = "",
    var age: Int = 0,
    var job: String = ""
)

fun main() {
    // before
    val person = Person()
    person.name = "정지원"
    person.age = 20
    person.job = "Android Developer"
    
    // after
    val person = Person().apply {
        name = "정지원"
        age = 20
        job = "Android Developer"
    }
}
```

### run

```kotlin
public inline fun <T, R> T.run(block: T.() -> R): R {
    contract { callsInPlace(block, InvocationKind.EXACTLY_ONCE) }
    return block()
}
```

- apply와 똑같이 동작하지만, run 코드블럭의 마지막 라인을 반환함.
- 수신객체에 대해 특정한 동작을 수행한 후 결과값을 리턴 받아야 할 경우에 사용.

```kotlin
data class Person(
    var name: String = "",
    var age: Int = 0,
    var job: String = ""
)

fun myJob(job: String): String = if(job != "") "백수아님!" else "백수임"

fun main() {
    val person = Person("지원", 20, "안드로이드 개발자")
    val isJobLess = person.run {
        job = ""
        myJob(job)
    }
    println(isJobLess)
    println(person)
}

실행결과
백수임
Person(name=지원, age=20, job=)
```

### with

```kotlin
public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    contract { callsInPlace(block, InvocationKind.EXACTLY_ONCE) }
    return receiver.block()
}
```

- 수신객체에 대한 작업 후 마지막 라인을 반환함.
    - run과 똑같이 동작
- 반환된 결과를 사용할 필요가 없는 경우 사용.
- 확장 함수가 아니며, 람다 내부 수신자는 this로 참조 가능함.

```kotlin
fun main() {
    val numbers = arrayListOf("1", "2", "3")
    val firstAndLast = with(numbers) {
        "첫 번째 요소 : ${first()}" + " 마지막 요소 : ${last()}" // 객체의 함수 호출
    }
    println(firstAndLast)
}
```

### let

```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract { callsInPlace(block, InvocationKind.EXACTLY_ONCE) }
    return block(this)
}
```

- 람다식의 마지막 라인을 반환함.
- 메서드 체인 결과에 대해 하나 이상의 함수를 호출하는데 사용할 수 있음.
- null이 아닌 값을 포함하는 코드 블록을 실행하는데 자주 사용됨. safe call과 같이 사용
- run과의 차이점은
    - run : this를 사용하여 호출하는 객체에 접근
    - let : it를 사용하여 호출하는 객체에 접근 → 람다의 인자

```kotlin
val str: String? = "Hello"   
//processNonNullString(str) // compilation error: str can be null
val length = str?.let { 
    println("let() called on $it")        
    processNonNullString(it) // OK: 'it' is not null inside '?.let { }'
    it.length
}

let() called on Hello
5
```

### also

```kotlin
public inline fun <T> T.also(block: (T) -> Unit): T {
    contract { callsInPlace(block, InvocationKind.EXACTLY_ONCE) }
    block(this)
    return this
}
```

- 수신 객체를 it로 사용하며, 수신 객체 자체를 반환함.
- 수신 객체 인자를 사용하는 일부 작업에 대해 유용함.
    - 객체에 대한 참조가 필요한 작업
- 디버깅이나 사이드 이펙트를 발생시키는 작업에 유용함.
- 코드에서 also는 “객체에 대해 다음을 수행함”이라고 읽음.
- Flow의 onEach랑 비슷함.

```kotlin
val numbers = mutableListOf("one", "two", "three")
numbers
    .also { println("The list elements before adding new one: $it") }
    .add("four")
```

### inline

- 람다의 경우 컴파일 단계에서 파라미터 개수에 따라 FunctionN 형태의 인터페이스로 변환됨.
    - 이는 일반 함수 구현에 비해 부가적인 비용이 들음.
- `inline` 함수는 호출된 곳에 함수의 본문의 코드를 삽입함.
    - 이는 바이트 코드로 바꿔줌.
- `inline` 코드를 남발하면 코드 라인 수가 증가하는 문제가 발생하여 적절하게 사용해야함.

```kotlin
inline fun <T> List<T>.customForEach(action: (T) -> Unit) {
    for (item in this) {
        action(item)
    }
}

fun main() { 
    val names = listOf("Alice", "Bob", "Charlie")
    names.customForEach { println(it) }
}
```

### nolnline

- `noinline` 은 람다 표현식 등을 인라인으로 호출한 곳에 코드를 삽입하지 않을 때 사용함.
    - 특정 람다만 인라인 처리를 원하지 않은경우에 사용함.

```kotlin
inline fun foo(inlinedLambda: () -> Unit, noinline notInlinedLambda: () -> Unit) {
    // ...
}
```

### crossinline

- 다른 함수의 고차함수 파라미터를 block {} 형태로 사용하는 경우 `crossinline`을 써야 함.
- 코틀린은 함수, 익명 함수를 종료할 때만 return을 사용할 수 있음.
    - 람다를 종료하려면 레이블을 사용해야함.
- 람다를 둘러싸는 함수를 반환할 수 없으므로, 람다 내부에서는 리턴이 금지됨.
- 람다에 위치하지만 둘러싸는 함수를 종료하는 것을 `비지역 반환`이라고 함.
    - `crossinline` 키워드는 `inline` 람다에서 비지역 반환을 금지함.

```kotlin
// 람다 반환 
fun ordinaryFunction(block: () -> Unit) {
    println("hi!")
}
fun foo() {
    ordinaryFunction {
        return // ERROR: cannot make `foo` return here
    }
}
fun main() {
    foo()
}
```

```kotlin
// 인라인 함수 람다 반환
// 람다가 전달되는 함수에 인라인이 있는 경우 반환값도 인라인이 될 수 있음. -> return 허용
inline fun inlined(block: () -> Unit) {
    println("hi!")
}
fun foo() {
    inlined {
        return // OK: the lambda is inlined
    }
}
fun main() {
    foo()
}
```

```kotlin
inline fun runOnThread(crossinline block: () -> Unit) {
    Thread {
        block()
    }.start()
}

fun main() {
    runOnThread {
        println("This is run on a new thread!")
        // return  이렇게 사용하면 컴파일 오류가 발생함. crossinline으로 비지역 반환을 금지시킴
    }
}
```

- block 람다에서 return을 허용한다면 main 함수에 조기 종료 시킬 위험이 있음.
- crossinline을 사용하여 비지역 반환을 금지하여 람다가 다른 스레드에 안전하게 실행되도록 보장함.

### 질문

1. p.383 커링 함수를 적용했을 때 이점은 가독성 부분일까요~? 그리고 커링함수를 어떤 상황에서 사용하면 좋을까요?.?
    1. 가독성이라 생각한 부분 : 여러 인자를 하나의 인자로만 받음.
        1. 인자가 많아질 경우 하나만 사용하므로 가독성이 향상될 거라 생각함.
2. p. 388 ~ 389 함수 연속 호출 부분에서

```kotlin
fun main() {
    val lamda = { x: Int -> { y: Int -> { z: Int -> x + y + z }}}
// val lamda = { x: Int, y: Int, z: Int -> x + y + z }
    println(lamda(100)(200)(300)) // 600
}
```

위 같은 람다 표현식은 가독성 측면에서 안좋아 보이는데, 특별히 사용되는 상황이 있을까요..?.?

3. p.409 SAM 인터페이스 사용시 이점이 있을까요?.? 
    1. 인터페이스 사용시 구현 해야하는 추상 메서드가 하나일 때 구현하는 코드를 작성 안해줘도 되므로, 가독성을 위한 것인지 궁금합니다!

```kotlin
fun interface StringSAMable {
    fun accept(s: String)
}

fun main() {
    val consume1 = StringSAMable { print(it) }.apply {
        accept("SAM!?!?")
    }
}
```

4. 인라인 함수를 사용하는 기준산정을 어떤 방식으로 해야할까요?.?
    1. 람다인 경우에는 무조건 사용해야할까요!? 만약 인라인 함수의 코드 라인 수가 많은 경우가 궁금합니다!

# 멘토님 답변
- 인라인 함수 사용하는 케이스는 콜스택 줄이기 위해 많이 사용.
  - 컴파일러에서 린트를 알려줌.
    - 코드 라인이 많은 경우
- with vs apply
  - with 완성된 객체가 있고 그 객체의 값을 작업할 때
  - apply 인스턴스를 만들면서 인스턴스를 가지고 작업을 할 때
    - 해당 인스턴스의 추가작업!
- run
    - 확장함수인 run보단 일반 run을 많이 사용함.
        - 코드블록을 만들어서 실행할 때 많이 사용됨.
            - ex) Elvis 마지막 항에 run을 사용하여 코드블록처럼 사용.
            - `str?.let { .. } ?: run { ... }`
- 2, 3번은 질문한 부분이 맞음.
- 1번은 함수형 프로그래밍 관련된 부분임.
- 스코프 함수에서 this가 체이닝 되는건 좋지 않음. ex) `run{..}.run{..}`
- 널 체크도 let으로 많이하지만, 조금 문제점이 있음.
  
```kotlin
val nickname = name?.let {} ?: "empty"
name이 null이거나, let 코드블록이 null인 경우 nickname에 empty가 들어감.
```
함수 체이닝 or 확실한 경우아니면 
```kotlin
val nickname = if ( name != null ) ?? else "emtpy"
```
이 방법을 지향해보자.
