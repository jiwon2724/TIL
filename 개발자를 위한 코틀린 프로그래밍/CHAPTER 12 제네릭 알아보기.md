# CHAPTER 12 : 제네릭 알아보기

### 제네릭

- 자료형을 특정 문자로 지정해서 타입 매개변수와 타입 인자로 사용하는 것
- 함수나 클래스 등을 제네릭으로 사용하면 사용 시점에 다양한 자료형을 처리해서 사용할 수 있음.

### 타입 매개변수와 타입 인자를 지정하는 위치

- 타입 매개변수는 `<>` 괄호 안에 하나 이상 정의할 수 있음. 보통 대문자 T, R, P, U를 사용
- 함수, 확장함수, 확장속성은 `fun`, `val/var` 다음에 `<>` 괄호를 사용하여 타입 매개변수 작성
- 클래스, 인터페이스, 추상 클래스는 이름 다음에 `<>` 괄호를 사용하여 타입 매개변수 작성

### 제네릭 함수

- 함수의 매개변수와 반환 타입을 일반 문자(T, R, P, U)로 지정해서 정의한 함수
- 함수가 호출될 때 해당 자료형을 인자로 지정하면 지정한 타입의 함수를 호출한 결과를 반환함.

```kotlin
fun main() {
    println(add(1, 2) { x, y -> x + y })
    println(add("1", "2") { x, y -> x + y })
}

fun <T> add(x: T, y: T, op: (T, T) -> T): T = op(x, y)

실행결과
3
12
```

### 타입 매개변수의 매개변수와 반환 자료형 분리

- 입력과 반환하는 결과가 다른 자료형일 경우 반환하는 자료형을 분리해서 타입 매개변수를 지정할 수 있음.
- 보통 반환 자료형의 타입 매개변수는 R로 표시함.

```kotlin
fun main() {
    println(sum(1, 2) { x, y -> "${(x + y)}!!!"})
    println(sum(1, 2) { x, y -> (x + y).toDouble()})
}

fun <T, R> sum(x: T, y: T, op:(T, T) -> R): R = op(x, y)

실행결과
3!!!
3.0
```

### 타입 매개변수에 특정 자료형을 제한하기

- 제네릭 함수의 타입 매개변수에 특정 자료형으로 처리하도록 제한할 수 있음.
    - 제한은 콜론을 붙인 후 특정 자료형을 적음.
- 지정된 자료형과 그 하위 자료형만 타입 인자로 처리할 수 있음.
- 복수개의 타입을 제한하려면 `where` 키워드를 붙이면 됨.

```kotlin
import java.lang.Appendable

fun main() {
    sumA("2", "1") { x, y -> x + y } // 에러 숫자 타입만 처리 가능
    sumA(2, 1) { x, y -> x + y }  // ok
    val name = StringBuilder("사랑하자!")
    suffix(name)
    println(name) // 사랑하자! 코틀린
}

fun <T: Number> sumA(x: T, y: T, action: (T, T) -> T): T = action(x, y)
fun <T> suffix(str: T) where T: CharSequence, T: Appendable {
    str.append("코틀린")
}
```

### 타입 매개변수를 사용해서 확장함수 만들기

- 타입 매개 변수의 이름으로 리시버를 지정하고 함수의 매개변수나 반환자료형을 타입 매개변수로 지정하면 됨.
- 타입 매개변수의 매개변수와 반환 자료형 분리도 기본 제네릭 함수와 일치함.

```kotlin
fun main() {
    println(11.map { it + it })
    println(111.map2 { "1 $it 1" })
}

fun <T> T.map(block: (T) -> T): T = block(this)
fun <T, R> T.map2(action: (T) -> R): R = action(this)

실행결과
22
1 111 1
```

### 제네릭 클래스

- 클래스도 프로퍼티나 메서드의 자료형을 일반화 하면 다양한 클래스로 객체를 생성하는 효과가 있음.
- 지정한 타입 매개변수는 프로퍼티의 자료형이나 메서드의 매개변수, 반환 타입에 사용됨.
    - 내부 속성이 타입 인자에 따라 다양한 종류의 객체를 만들 수 있음.

```kotlin
fun main() {
    val company = Company("인공지능")
    val company2 = Company2<String>("인공지능")
}

class Company(text: String) {
    var x = text
    init { println("초기화 $x") }
}

class Company2<T>(text: T) {
    var x = text
    init { println("초기화 $x") }
}

실행결과
초기화 인공지능
초기화 인공지능
```

### 제네릭 인터페이스

- 인터페이스가 작업하는 타입을 파라미터화 할 수 있음.

```kotlin
interface Repository<T> {
    fun getById(id: Int): T?
    fun getAll(): List<T>
}

data class User(val id: Int, val name: String)

class UserRepository : Repository<User> {
    private val users = listOf(
        User(1, "Alice"),
        User(2, "Bob"),
        User(3, "Charlie")
    )

    override fun getById(id: Int): User? {
        return users.find { it.id == id }
    }

    override fun getAll(): List<User> {
        return users
    }
}

fun main() {
    val userRepository = UserRepository()
    println(userRepository.getById(1))
}

실행결과
prints: User(id=1, name=Alice)
prints: [User(id=1, name=Alice), User(id=2, name=Bob), User(id=3, name=Charlie)]
```

### 변성(Variance)

- 제네릭으로 정의하면 해당 제네릭에 매칭되는 자료형만 대체되어 처리됨.
- 상속관계까지 처리되려면 변성을 지정해야함.
    - 변성은 제네릭 타입의 관계를 설명하는 개념임.
    - 공변성(Covariance), 반공변성(Contravariance), 무변성(Invariance)이 있음.

### 공변성(Covariance)

- `out` 키워드를 사용하여 표시됨.
- 타입 Producer<out T>가 있을 때 T의 하위 타입은 Producer의 하위 타입으로 간주됨.
- read-only 상황에서 안전하게 사용됨.
    - 타입파라미터 T를 매개변수로 받아들이지 못함.
        - 공변성은 생산위치에서만 사용이 가능하고, 소비 위치에선 사용될 수 없음.
        - 즉, 파라미터로 들어온 값은 `get`만 가능하고, `set`은 불가능함.

```kotlin
open class Fruit
class Apple : Fruit()
class Orange : Fruit()

class Box<out T>(private val item: T) {
    fun get(): T = item
    // 아래 함수는 에러를 발생시킴.
    // fun set(newItem: T) { ... }
}

fun main() {
    val appleBox: Box<Apple> = Box(Apple())
    val fruitBox: Box<Fruit> = appleBox  // 공변성 덕분에 가능
    val fruit: Fruit = fruitBox.get() // (생산)

    // 만약 set 함수가 허용된다면 아래와 같은 상황이 발생할 수 있음.
    // fruitBox.set(Orange()) // (소비) Apple만 들어갈 수 있는 appleBox에 Orange가 들어가는 상황
}
```

### 반공변성(Contravariance)

- `in` 키워드를 사용하여 표시됨.
- 타입 Consumer<in T>가 있을 때, T의 상위 타입이 Consumer의 하위 타입으로 간주됨.
    - 즉, 하위 타입 관계가 반대로 유지되는 것.
    - A가 B의 하위타입 이라면 C<B>가 C<A>의 하위 타입이다.
- write-only 상황에서 안전하게 사용됨.
    - 소비만 가능함. → `set`

```kotlin
open class Animal
class Cat : Animal()

interface Consumer<in T> {
    fun consume(item: T)
}

class AnimalConsumer : Consumer<Animal> {
    override fun consume(item: Animal) = println("Consumed an animal")
}

fun main() {
    val catConsumer: Consumer<Cat> = AnimalConsumer()
    catConsumer.consume(Cat()) // 소비
}
```

### 무변성(Invariance)

- 타입 파라미터 T는 변하지 않음. 즉, 그대로 유지됨
- 기본적으로 모든 제네릭 타입은 무변성임.

```kotlin
class Box<T>(var value: T)

fun main() {
    val intBox: Box<Int> = Box(1)
    val anyBox: Box<Any> = Box(Any())

    // 에러
    // val anotherBox: Box<Any> = intBox 
}
```

- `Box<Int>`는 `Box<Any>`로 할당될 수 없음. 이는 `Int`와 `Any`의 관계에 상관없이 `Box<T>`의 무변성 때문임.

### 변성 정리

![image](https://github.com/jiwon2724/TIL/assets/70135188/d450c8f9-d159-4ae9-b74d-32571aefd5b9)


- `out` (공변성)은 타입 파라미터가 반환 값으로만 사용될 수 있어 "생산"하는 측면에서 사용됨.
- `in` (반공변성)은 타입 파라미터가 함수의 매개변수로만 사용될 수 있어 "소비"하는 측면에서 사용됨

### 리플렉션(Reflection)

- 런타임에 프로그램의 클래스, 함수, 프로퍼티 등을 조사하기 위해 사용 되는 기술임.
- 프로그램이 실행중일 때 인스턴스 등을 통해 객체의 내부 구조 등을 파악할 수 있음.
- 더블 콜론(`::`)을 사용하여 참조할 수 있음.

### 주요 리플렉션 기능

- **KClass**
    - 클래스에 대한 참조를 나타냄.
    - 코틀린 Class에 해당하는 리플렉션 객체임.
    - 클래스의 이름, 속성, 함수, 객체, 생성자 등과 같은 클래스에 관련 정보에 접근할 수 있음.

```kotlin
val stringClass: KClass<String> = String::class
```

- **KFunction**
    - 함수에 대한 참조를 나타냄.
    - 메서드나 함수의 파라미터, 반환 타입 등 함수 관련 정보에 접근할 수 있음.

```kotlin
fun greet(): Unit { println("Hello") }
val funRef: KFunction<Unit> = ::greet
```

- **KProperty**
    - 프로퍼티에 대한 참조를 나타냄.
    - `KProperty1`, `KProperty2` 등과 같이 인덱스가 붙은 서브타입이 있으며, 이는 프로퍼티의 수 (getter의 파라미터 수)를 나타냄.
        - ex) Pair, Triple, 구조분해 등
    - 프로퍼티의 타입, 이름 등 프로퍼티 관련 정보에 접근할 수 있음.

```kotlin
val x: Int = 10
val propRef: KProperty<Int> = ::x
```

- **타입 파라미터와 리플렉션**
    - 타입 파라미터의 `KType`을 얻으려면, `typeOf` 함수와 `reified` ****제네릭을 사용.

```kotlin
inline fun <reified T> getType() = typeOf<T>()
```

- **생성자 참조**
    - 더블 콜론과 함께 클래스의 이름을 사용.

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val constructor = ::Person
    val person = constructor("Alice", 29)
}
```

### 애노테이션

- 프로그램 코드에 특정 주석을 부여해 개발 툴이나 JVM에 정보를 추가하는 것.
- `@+애노테이션` 이름을 붙여서 사용함.
- 코드의 동작에 직접적인 영향을 주지 않지만, 다른 코드나 도구가 해당 코드를 어떻게 처리해야 하는지에 대한 정보나 지침을 제공할 수 있음.

```kotlin
@Deprecated("Use newFunction instead.", ReplaceWith("newFunction()"))
fun oldFunction() {
    // ...
}

fun newFunction() {
    // ...
}
```

### 질문

1. 클래스를 인스턴스화 해서 만들어진 객체에 해당 클래스의 속성, 함수 등을 참조할 수 있는데 리플렉션을 사용해 특별히 참조해주는 이유가 있나요!?
    1. 리플렉션을 사용해서 각 정보에 접근해야 하는 상황은 언제인지 알 수 있을까요?.?
2. 런타임 시 제네릭은 타입소거가 되는 것으로 알고있어요. 타입 파라미터를 알아야 하는 경우엔 reified를 사용해서 처리해주는게 맞나요?
    1. 타입 파라미터를 알아야 하는 경우도 빈번하게 일어아는 경우인지도 궁금합니다.
3. 제네릭의 변성은 안드로이드에서 보통 타입에 대한 정확성, 안정성, 유연성을 보장할 때 사용될까요?

```kotlin
sealed interface Result<out T> {
    data class Success<T>(val data: T) : Result<T>
    data class Error(val exception: Throwable) : Result<Nothing>
}

suspend fun <T : Any> setResult(response: (suspend () -> T)): Result<T> {
    return try {
        Result.Success(response.invoke())
    } catch (e: Exception) {
        Result.Error(e) // out을 제거하면 해당 라인 에러. Nothing은 모든 클래스의 하위타입.
    }
}
```

위 같은 규칙을 정의할 때 많이 사용되는지 궁금합니다.
