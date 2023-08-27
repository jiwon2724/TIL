# CHAPTER 07 : 클래스 연간관계 알아보기

### 클래스 관계

- 상속관계(`is-a`) : 클래스를 상속해서 하나의 클래스 처럼 사용.
- 연관관계(`has-a`) : 클래스를 상속하지 않고 내부적인 속성에 객체를 만들어서 사용.
- 결합관계(`약한 has-a`) : 연관관계를 구성하는 방식 중 클래스 간 주종관계 없이 단순하게 사용하는 관계.
- 조합관계(`강한 has-a`) : 연관관계를 구성하는 방식 중 클래스 간의 주종관계가 있어 따로 분리해서 사용할 수 없는 관계

### 결합(Aggregation)관계

- 약한 has-a 관계임
- 단순하게 사용될 클래스의 객체를 속성으로 만들어 사용함.
    - 보통 주 생성자에 객체를 전달받아 구성
- 클래스간 동일한 생명주기일 필요는 없음.
    - 즉, 한 클래스의 객체가 소멸해도 다른 클래스의 객체는 계속 활용할 수 있음.

```kotlin
fun main() {
    val collegeAddress: Address = Address(country = "대한민국", "관악구")
    val college: College = College("서울대", collegeAddress)
    println(college)
}

data class Address(
    val country: String,
    val city: String

data class College(
    val collegeName: String,
    val collegeAddress: Address
)

실행결과
College(collegeName=서울대, collegeAddress=Address(country=대한민국, city=관악구))
```

### 조합(Composition)관계

- 결합관계보다 더 결합이 강한 관계임.
- 두 클래스는 주종 관계이고, 생명주기도 동일함.
    - 즉, 두 클래스는 항상 같이 생성되고, 같이 소멸하는 구조일 경우에만 사용함.

```kotlin
fun main() {
    val myCar = Car(
        color = "Red",
        maxSpeed = 3000,
        name = "Ray"
    )
    myCar.carEngine = CarEngine()
    
    myCar.run {
        carEngine.startEngine()
        carInfo()
        carEngine.stopEngine()
    }
}

class CarEngine {
    fun startEngine() { println("엔진 가동") }
    fun stopEngine() { println("엔진 중단") }
}

class Car(val color: String, val maxSpeed: Int, val name: String) {
    lateinit var carEngine: CarEngine // 필드 주입

    fun carInfo() { println("$color $maxSpeed $name") }
}

실행결과
엔진 가동
Red 3000 Ray
엔진 중단
```

### 의존(Dependency)관계

- 한 클래스가 다른 클래스의 기능이다 행동에 의존하고 있을 나타내는 관계임.

```kotlin
class CPUx86() {
    fun process() { ... }
}

class Computer() {
    val cpu = CPUx86()

    fun start() {
        cpu.process() { ... }
    }
}
```

- Computer 클래스는 CPUx86 클래스에 의존하고 있음.
- Computer 클래스는 CPUx86 클래스 생성에 대한 책임을 가지고있음.
- 위 코드는 높은 결합도를 유지하고 있음.
    - CPUx86 클래스의 변경에 Computer 클래스도 변경됨.
        - 즉, 높은 결합도는 작은변경에도 많은 영향을 주는 것임.

# 속성과 메서드 재정의

- 코틀린의 속성을 일반적인 변수처럼 정의하고 사용함.
- 속성으로 사용하려면, 게터와 세터를 재정의해서 사용.

```kotlin
fun main() {
    val kClass = Kclass()
    kClass.attr
    kClass.attr2 = 2
}

class Kclass {
    val attr: Int = 0
        get() {
            println("attr get()")
            return field
        }

    var attr2: Int = 0
        get() {
            println("attr get()")
            return field
        }
        set(value) {
            println("attr2 set($value)")
            field = value
        }
}

실행결과
attr get()
attr2 set(2)
```

### 연산자 오버로딩

- `operator` 예약어를 사용해서 연산자에 해당하는 메서드를 재정의 할 수 있음.

```kotlin
// 클래스 내부 연산자 오버로딩
fun main() {
    val kClass = Kclass()
    val strNumber = kClass + 300
    println(strNumber) // 문자열임 300
}

class Kclass {
    operator fun plus(other: Int): String = ("문자열임 $other")
}

// 확장함수 연산자 오버로딩
fun main() {
    println("재정의" * 3) // 재정의재정의재정의
}

operator fun String.times(other: Int): String {
    var str = "" 
    repeat(other) { str += this }
    return str
}
```

- 기본타입은 확장함수로 재정의 할 수 없음.
    - `Int`, `Float`, `Double` 등

### infix 처리 (중위 표현법)

- 클래스의 멤버 호출이나, 확장함수 사용시 사용하는 점`(.)`을 생략하고 함수 이름 뒤에 소괄호를 생략해 직관적인 이름을 사용할 수 있는 표현법임.
    - 멤버 메서드 또는 확장 함수여야 함.
    - 하나의 매개변수를 가져야 함.
    - `infix` 키워드를 사용하여 정의해야 함.

```kotlin
fun main() {
    val nomalMulti = 3.multiply(10)
    val infixMulti = 3 multiply 10 // 연산자 처럼 사용됨.
    println(infixMulti) // 30
}

infix fun Int.multiply(x: Int): Int { // infix로 선언되므로 중위 함수임.
    return this * x
}
```

# 특정 자료를 다루는 클래스 알아보기

### 데이터 클래스

- 클래스는 행위를 중심으로 데이터를 은닉함
- 클래스 간 전송, 주입 등을 하려면 데이터 속성만 가진 클래스가 필요한데, 이때 데이터 클래스를 정의해서 사용하면 편리함.

```kotlin
data class Client(val name: String, postalCode: Int)
```

- 데이터 클래스는 내부적으로 `toString` , `eqauls` , `hashCode` 를 가지고 있음.
    - `eqauls` 와 `hashCode` 는 주 생성자에 나열된 모든 프로퍼티를 고려해 만들어지고, 모든 프로퍼티 값의 동등성을 확인함.
        - `hashCode` 는 모든 프로퍼티의 해시 값을 바탕으로 계산한 해시 값을 반환함.
            - hashCode : 객체를 식별할 수 있는 값을 반환.
    - 주 생성자 밖에 정의된 프로퍼티는 `equals` 나 `hashCode` 를 계산할 때 고려의 대상이 아님.
- 위 내용을 바탕으로 데이터 클래스의 동등성 비교를 쉽게 가능함.

```kotlin
fun main() {
    val client1 = Client("정지원", 1234)
    val client2 = Client("정지원", 1234)
    println(client1 == client2) // true
    println(client1 === client2) // false
}

fun main() {
    val client1 = Client("정지원", 1234)
    val client2 = client1
    val client3 = client1
    println(client2 === client3) // true
}

data class Client(val name: String, val postalCode: Int)
```

### 이넘 클래스(Enum Class)

- 여러 상수값을 그룹화하고, 각 상수 값이 해당 그룹의 모든 값 중 하나임을 보장함.
    - 즉, 특정 상숫값을 관리하는 클래스
- 이넘 클래스 안에서도 프로퍼티와 메서드를 정의할 수 있음.
- 코드의 가독성과 유지보수성이 향상됨.

```kotlin
fun main() {
    println(CardType.PLATINUM.ordinal) // 객체의 순서 2
    println(CardType.PLATINUM.name) // 객체의 이름 PLATINUM
}

enum class CardType {
    SILVER, COLD, PLATINUM
}
```

### 이넘 클래스의 속성, 함수 추가와 when처리

```kotlin
fun main() {
    println(INDIGO.rgb())
    println(getMnemonic(INDIGO))
}

enum class Color(
    val r: Int,
    val g: Int,
    val b: Int
) {
    RED(255, 0, 0), ORANGE(255, 165, 0),
    YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255),
    INDIGO(75, 0, 130), VIOLET(238, 130, 238);

    fun rgb() = (r * 256 + g) * 256 + b
}

fun getMnemonic(color: Color) : String =
    when(color) {
        RED -> "Richard"
        ORANGE -> "Of"
        YELLOW -> "warm"
        BLUE, INDIGO -> "cold"
        GREEN, VIOLET -> "midnight"
    }

실행결과
4915330
cold
```

### 인라인 클래스(value 클래스)

- 내부에 하나의 프로퍼티만 선언한 특수한 형태의 클래스임.
- 코틀린 1.5.0 이후부터 value 클래스로 바뀜.

```kotlin
@JvmInline
value class Password(private val s: String) // Password의 wrapper 클래스

런타임에 String으로 사용됨.
```

- 위 처럼 선언하면 Password 클래스는 inline class가 됨.
    - value class
    - `@JvmInline` 어노테이션은 컴파일 타겟이 JVM일 경우에 붙임.
- 단순한 wrapper 클래스를 만들어 사용하는건 성능면에서 안좋음.
    1. 새로운 인스턴스로 인한 heap 영역 낭비
    2. 런타임 primitive 타입 최적화 X
        1. 런타임에 그 래퍼 객체의 인스턴스가 실제로 생성되어 메모리를 차지하게 됨.

```kotlin
기본 유형은 일반적으로 런타임에 의해 크게 최적화되는 반면, 래퍼는 특별한 처리를 받지 않음.
```

- 위 같은 성능 문제로 인라인 클래스(value 클래스)는 런타임시 객체를 인스턴스화하지 않는 방식을 사용함.

### 질문

p.258 

특정 클래스가 삭제된다는건 어떤 뜻인가요? 클래스와 소멸과 삭제의 정의가 궁금합니다! 

- 클래스 내부에서 클래스를 생성 → 소멸과 삭제가 같이됨.
- 클래스 외부에서 주입하는 행위 → 별개임.

`멘토님 답변` : 위 표현이 맞음

p.259

“실제 비지니스상에서는 상속관계와 마찬가지로 이런 의미의 구성은 별로 발생하지 않는다.”

라고 나와있는데, 컴포지션은 has-a 관계일 땐, 많이 사용하는 것으로 알고 있는데 책에 나와있는 내용이 맞을까요~?


인라인 클래스(value class)

1. 코틀린 문서에서는 다음과 같은 문장이 나오는데요! `비즈니스 로직이 일부 유형에 대한 래퍼를 생성해야 하는 경우` 이런 경우는 어떤 경우인가요~!?!??
2. 인라인 클래스를 사용하는 이유는 wrapping된 클래스의 런타임 시 성능적인 부분의 이슈를 해결하기 위해 사용되는게 맞을까요?
   - 런타임적 성능을 알기위해선 JVM 내부 구조를 파악해야하는게 맞을까요~?
3.

```kotlin
@JvmInline
value class Password(private val s: String)
```
위 처럼 사용되는 경우는 어떤 경우가 있을까요..?

`멘토님 답변` : JVM 내부 구조를 파악하는 것은 도움이됨. 위 같은 경우는 런타임시 Password를 만들지 않고 String처럼 사용하게 됨.
이는 성능개선에 도움이 됨.

공통 답변 : 
- 코틀린 1.9.0 이후부턴 Enum Class에서 `values()`대신 `entries`를 사용하는 것을 지향함. 성능부분에서 개선됨.
- Enum Class에서 `ordinal` 사용을 지양할 것.
  - Enum Class의 값을 추가, 삭제 시 번거로움
- 1.9.0 부터 value class에 보조 생성자를 사용할 수 있음.
- 1.9.0 부터 data object라는 것이 생김.
  - compoentN이 없음
  - copy 없음.
  - 프로퍼티를 가질 수 없음.
  - `object`의 성격을 나타내는 데이터임.
