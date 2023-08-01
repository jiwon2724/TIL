# CHAPTER 04 : 함수 알아보기

# 함수 알아보기

### 함수 정의와 실행

- 함수를 사용하려면 먼저 함수를 정의해야함.
- 한번 정의한 함수는 여러 번 호출하여 사용가능함. 이는 중복하는 코드를 줄여줌.
- 함수의 정의는 머리부, 몸체부로 구분함.

```kotlin
// 반환값이 있는 함수
fun add(num1: Int, num2: Int): Int {
    val result = num1 + num2
    return result
}

// 단일 표현식으로 대체
fun add(num1: Int, num2: Int): Int = num1 + num2

// 반환값이 없는 함수
fun printMessage() {
    print("Message")
}
```

- 머리부
    - `fun` : 함수를 정의하는 예약어 function의 약자임. 함수 이름 앞에 붙여야함.
    - `add` : 함수의 이름. `fun` 다음에 함수 이름을 작성.
        - 위 예시에서 함수의 이름이 add임. 이는 개발자가 정의하는 함수의 이름임.
            - 함수 이름은 영어 소문자로 시작함.
    - `(num1: Int, num2: Int)` : 함수 내부에서 사용되는 매개변수임.
    - `: Int` : 리턴 값 표시. 해당 타입에 대한 값을 반환함.
- 몸체부
    - 함수 몸체부 : `{ .. }` 중괄호 코드 블록에 작성함.
    - 지역 변수 : `val result = num1 + num2` 함수 내부에 정의되는 변수. 함수가 종료되면 사라짐.
    - 반환값 처리 : `return result`

### 함수의 매개변수와 인자

- 함수의 파라미터에 초깃값(default value)을 지정할 수 있음.

```kotlin
fun defaultArg(x: Int =100, y: Int = 200) = x + y // 300
```

- 가변인자 지정
    - `vararg` 키워드를 사용하여 여러개의 인자를 전달 받을 수 있음.
    - 배열로 정의된 것을 가변인자로 처리하려면 `*` 스프레드 연산자를 사용해야함.
        - 스프레드를 처리할 땐 array등 기본 배열을 사용해야함.

```kotlin
fun addNum(vararg num: Int): Int {
    var result = 0
    for(i in num) result += i
    return result
}

val result = addNum(1, 2, 3, 4, 5) // 10
val intArray = intArrayOf(1, 2, 3, 4, 5)
val spreadResult = addNum(*intArray)
```

- 함수 인자를 전달할 때 주의 사항
    - `mutable` 한 데이터를 파라미터로 전달한 경우 함수 밖에 정의한 값도 같이 변경됨.
    - 함수 내부에서 외부의 값을 변경하지 않으려면 `immutable` 한 값으로 설정하거나, `mutable` 한 값을 복사해서 파라미터에 전달해야함.

```kotlin

fun main(args: Array<String>) {
    val immutableList = listOf(1, 2, 3, 4, 5)
    val mutableList = mutableListOf(1, 2, 3, 4, 5)

    addImmutableList(immutableList)
    addMutableList(mutableList)
    // addMutableList(mutableList.toMutableList()) 값 복사

    println("immutableList: ${immutableList.toString()}") // 1, 2, 3, 4, 5
    println("mutableList: ${mutableList.toString()}") // 1, 2, 3, 4, 5, 6
}

fun addImmutableList(immutableList: List<Int>) = immutableList + listOf(6, 7)
fun addMutableList(mutableList: MutableList<Int>) = mutableList.add(6)
```

# 익명 함수와 람다 표현식 알아보기

### 익명함수

함수 정의와 동일하지만 이름을 가지지 않음. 일회성으로 처리하는 요도로 사용.

```kotlin
val addResult = fun (a: Int, b: Int) = a + b
printf(addResult(100, 200)) // 300

// 익명함수 즉시 실행
val addResult = fun (a: Int, b: Int): Int { return a + b }(100, 200)
println(addResult) // 300
```

### 람다 표현식

- 수학에서 상수를 의미함. 즉, 상수처럼 사용하는 함수를 의미함.
- 익명함수 보다 함수를 정의하고 상수처럼 인자나 반환값 등을 전달하기 편리함.
- 다른 함수에 넘길 수 있는 작은 코드 조각을 뜻함.
- 람다 표현식 표기법
    - 예약어와 함수 이름이 없음
    - 중괄호 안에 직접 매개변수와 표현식을 작성
    - 매개변수와 표현식을 구분하는 기호 `->` 로 구분

```kotlin
{ println("아무 인자 없는 람다") } // 아무 인자 없는 람다
println({x: Int -> x + x}(10)) // 인자가 하나인 경우 -> 100
val result = { x: Int, y: Int -> x + y } // 재사용시 변수에 할당
println(result(1, 2)) // 3
println(highOrderFunction(1, 2) { x, y -> x * y }) // 2

// 함수를 인자로 받는 함수
fun highOrderFunction(x: Int, y: Int, f: (Int, Int) -> Int) = f(x, y)
fun returnHighOrderFunction(): () -> Int = { 100 } // 람다를 반환하는 함수
```

- 함수를 파라미터로 받는 함수가 마지막에 온 경우엔 `,` 를 제외할 수 있음.

### 클로저

- 람다식으로 표현된 내부 함수에서 외부 범위에 선언된 변수에 접근할 수 있는 개념.
- 접근한 외부 변수를 람다식 안에서 값을 유지하기 위해 람다식이 포획(Capture)한 변수임.

```kotlin
fun closureExample(): () -> Unit {
    var count = 0 

    // 클로저
    return {
        count++
        println("Count: $count")
    }
}

fun main() {
    val closure = closureExample()

    closure() // Count: 1
    closure() // Count: 2
    closure() // Count: 3
}
```

### 함수참조

- 함수 자체를 값으로서 다룰 수 있게 해주는 기능.
- 더블콜론 `::` 을 사용하여 함수를 참조.
    - 코틀린에서 `::` 은 리플렉션임.
        - 리플렉션이란 코드를 작성하는 시점에는 런타임상 컴파일된 바이트코드에서 내가 작성한 코드가 어디에 위치하는지 알 수 없기때문에 바이트코드를 이용해 내가 참조하려는 값을 찾기위해 사용.
- 함수참조를 사용하면 함수를 변수에 할당하거나 다른 함수에 인자로 전달할 수 있음.
- 코드 재사용성과 가독성 향상을 위해 사용됨.
    - 고차함수로 파라미터와 리턴 값 지정 등

```kotlin
fun isOdd(x: Int) = x % 2 != 0

val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd)) // 1, 3
```

# 함수 자료형 알아보기

코틀린에서 함수는 1급 객체임. 따라서 함수는 변수에 할당이 가능하며, 함수의 인자로 전달가능 하고, 함수의 리턴값으로도 사용 할 수 있음.

### 변수에 함수 자료형 처리

```kotlin
fun main() {
    val a: () -> Unit = { println("함수") } // 매개변수와 반환 값이 없는 함수
    val b: (Int) -> Int = { x -> x * 3} // 하나의 매개변수로 처리하고 반환 값은 Int
    val c: (Int, Int) -> Int = { x, y -> x + y} // 두 개의 매개변수로 처리하고 반환 값은 Int

    a() // 함수
    println(b(10)) // 30
    println(c(10, 20)) // 30
}
```

위 람다식은 익명 함수로 치환이 가능함.

### 함수를 반환하는 함수를 변수에 할당

- 함수의 반환값이 함수라서 함수의 자료형을 지정해야함.

```kotlin
val innerFunc: (Int, Int) -> () -> Int = { x, y -> { add(x, y) }}
fun add(x: Int, y: Int): Int = x + y
println(innerFunc(1, 2)()) // 3
```

- 복잡하고 가독성이 좋지 않아서 많이 사용은 안할 것 같음.

### 널이 가능한 함수 자료형 정의

- Nullable 함수 자료형 : (함수 자료형)?
    - 함수 자료형 전체를 소괄호로 묶고 `?` 를 붙임.
        - 이는 파라미터나, 반환 타입에 해당함.
- Nullable 함수 자료형 함수 호출 : `?.invoke`
    - 널이 들어온 경우 null을 안전하게 처리하기 위해 `invoke` 사용

```kotlin
fun nullFunc(action: (() -> Unit)?): Long {
    action?.invoke() // action()
    return System.nanoTime() - start
}
```

### 호출메서드(invoke)

- `invoke` 메서드가 실행되면 함수의 반환 자료형에 속하는 객체를 반환함.
    - 자기 자신을 함수처럼 호출함.

### 클래스에 연산자 오버로딩

- `invoke` 함수를 오버라이딩 해서 사용할 수 있음.

```kotlin
class A : Function<Unit> { // 함수 자료형 인터페이스 상속
    operator fun invoke() {
        println("연산1")
    }
}

class MyFunction : () -> Unit { // 함수 자료형 상속
    override fun invoke() {
        println("연산2")
    }
}
```

### object 정의와 표현식으로 호출 연산자 처리

```kotlin
val a = object : (Int, Int) -> Int {
    override fun invoke(p1: Int, p2: Int): Int {
        TODO("Not yet implemented")
    }
}
```

- 하나의 객체만 만들게돼서 클래스에서 정의하는 것보다 더 많이 사용됨.
- 위 처럼 정의한 문장은 `익명객체` 라 부름. 함수 타입 뿐만 아니라, 인터페이스, 클래스가 올 수 있음.

### 함수 오버로딩 줄이기

- 여러 개의 함수를 정의하는 방식이 다 좋은 것은 아님.
- 함수의 오버로딩을 줄이려면 파라미터에 초깃값을 사용하거나 가변인자(`vararg`)로 변경해서 작성해야함.

# 질문

1. `invoke` 를 언제 사용해야하나요!? - 사용하는 상황이 있나요!?

단순 널처리에 대한 가독성 때문에 사용인지..?

```kotlin
fun nullFunc(action: () -> Unit): Long {
    action() // action.invoke()
    return System.nanoTime() - start
}

action() == action.invoke()
```

2. 함수 자료형 인터페이스와 함수 자료형 상속은 어떤 경우에 사용되나요?.?
