# CHAPTER 11 위임 확장 알아보기

### 클래스 위임

- 특정 클래스에 자기 클래스가 할 일을 다른 클래스에 맡겨 처리하는 것.
    - 위임관계는 패턴으로 지원함.
- 코틀린에선 `by`를 사용하여 위임관계를 쉽게 구성할 수 있음.
    - 이것을 클래스 위임(delegation)이라고함.

### 클래스 위임 처리

- 보통 하나의 클래스에는 하나의 책임을 부여해서 설계하는 방식을 사용함.
- 위임은 2개의 클래스가 동일한 책임을 가지고 나눠서 처리함.
    - 다양한 기능을 하나의 클래스를 통해서 받고 처리할 수 있도록 구조화함.

```kotlin
interface Base { fun say() }

class BaseImpl(private val x: Int) : Base {
    override fun say() { println("베이스 클래스 구현 : $x") }
}

class Derived(private val b: BaseImpl) : Base {
    override fun say() { b.say() }
}

class Derived2 : Base by BaseImpl(10)

fun main() {
    val b = BaseImpl(10)
    Derived(b).say() // 베이스 클래스 구현 : 10
    Derived2().say() // 베이스 클래스 구현 : 10
}
```

### 생성자의 매개변수 속성으로 위임 처리

- 주 생성자의 속성의 자료형을 인터페이스로 작성.

```kotlin
interface SoundBehavior { fun makeSound() }

class ScreamBehavior : SoundBehavior {
    override fun makeSound() = println("Screaming!!!")
}

class RockAndRollBehavior : SoundBehavior {
    override fun makeSound() = println("Rock'n'roll!!!")
}

class Person(soundBehavior: SoundBehavior) : SoundBehavior by soundBehavior

fun main() {
    Person(ScreamBehavior()).makeSound() // Screaming!!!
    Person(RockAndRollBehavior()).makeSound() // Rock'n'roll!!!
}
```

### 클래스 위임 활용

1. 내장된 집합 인터페이스를 위임 처리

```kotlin
class CounterSet<T>(
    private val innerSet: MutableSet<T> = mutableSetOf()
) : MutableSet<T> by innerSet {
    var elementAdded: Int = 0
        private set

    override fun add(element: T): Boolean {
        elementAdded++
        return innerSet.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        elementAdded += elements.size
        return innerSet.addAll(elements)
    }

    fun display(): MutableSet<T> = innerSet
}

fun main() {
    val counterList = CounterSet<String>()
    counterList.addAll(listOf("A", "B", "C", "D", "E"))
    println(counterList.elementAdded) // 5
    println(counterList.display()) // [A, B, C, D, E]
}
```

- 위임을 사용하지 않은 경우, MutableSet의 재정의 메서드를 모두 오버라이딩 해야함.
- 위임을 사용하여 CounterSet 클래스는 MutableSet의 인터페이스를 상속 받음.
    - 이 모든 구현은 innerSet에 위임됨.
    - `add`, `addAll` 함수는 재정의 됐으므로 제외.

```kotlin
💡 by 키워드는 CounterSet에 MutableSet의 모든 멤버 함수를 자동으로 복사하는 것처럼 동작함.
  실제로 복사되진 않고, 호출 시 위임 객체(innerSet)의 해당 함수가 호출됨.
```

2. 데이터 저장과 기능 관리를 분리한 클래스 위임
- 더 세부적으로 계층을 구조화할 수 있음.

```kotlin
// 잔액 관리 클래스
class Balance(val accountNo: Int, var balance: Int)

// 입, 출금 처리
interface Accountable {
    fun deposit(acc: Balance, amount: Int)
    fun withdraw(acc: Balance, amount: Int)
}

// 입, 출금 구현 클래스
class DWManager : Accountable {
    override fun deposit(acc: Balance, amount: Int) { acc.balance = acc.balance + amount }
    override fun withdraw(acc: Balance, amount: Int) { acc.balance = acc.balance - amount }
}

// 계좌 관리 클래스
class Agreement(
    val accountNo: Int,
    val dwManager: Accountable
) : Accountable by dwManager

fun main() {
    val dwManager = DWManager()
    val b = Balance(1, 0)
    val a = Agreement(1, dwManager)

    a.dwManager.deposit(b, 1000)
    println("계좌번호 : ${a.accountNo}")
    println("계좌번호 : ${b.accountNo} 잔액 : ${b.balance}")
    a.dwManager.withdraw(b, 500)
    println("계좌번호 : ${b.accountNo} 잔액 : ${b.balance}")
}

실행결과
계좌번호 : 1
계좌번호 : 1 잔액 : 1000
계좌번호 : 1 잔액 : 500
```

### 속성 위임

- 코틀린의 속성은 게터와 세터 접근자를 정의해서 실제 변수를 참조하는 것이 아닌 메서드를 참조해서 처리하는 방식임.
    - 그래서 속성에도 위임을 처리할 수 있음.

### 속성 변경 관찰

- `Delegates.observable`을 사용하여 속성이 변경되는 상태를 관찰할 수 있음.

```kotlin
import kotlin.properties.Delegates

class User {
    var name: String by Delegates.observable("<no name>") { _, old, new ->
        println("$old -> $new")
    }
}

fun main() {
    val user = User().apply { 
        name = "Test1"
        name = "Test2"
        name = "Test3"
        name = "Test4"
    }
}

실행결과
<no name> -> Test1
Test1 -> Test2
Test2 -> Test3
Test3 -> Test4
```

### 특정 조건이 일치할 때만 속성 위임 변경

- `Delegates.vetoable`을 사용함.

```kotlin
import kotlin.properties.Delegates

class User {
    var age: Int by Delegates.vetoable(0) { _, _, newValue ->
        newValue >= 0
    }
}

fun main() {
    val user = User()
    user.age = -1  // 변경이 거부됩니다.
    println(user.age)  // 0
    user.age = 30
    println(user.age) // 30
}
```

### 클래스를 만들어 속성 위임 처리

- 게터와 세터를 구성하는 방식을 별도의 클래스로 정의해 객체를 생성해서 처리하는 구조.
- getValue, setValue 메서드를 정의해야함.
    - 위 메서드들은 기본으로 객체의 레퍼런스와 속성을 가짐.
        - 속성은 KProperty로 처리됨.

```kotlin
💡 KProperty는 속성에 대한 참조를 나타냄. 런타임에 속성의 정보를 검색하거나 속성에 접근할 수 있음.
```

```kotlin
import kotlin.reflect.KProperty

class NonNegativeIntDelegate {
    private var value: Int = 0

    operator fun getValue(thisRef: Any?, property: KProperty<*>): Int {
        return value
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, newValue: Int) {
        if (newValue < 0) {
            throw IllegalArgumentException("Age cannot be negative!")
        }
        value = newValue
    }
}

class Person {
    var age: Int by NonNegativeIntDelegate()
}

fun main() {
    val person = Person()
    person.age = 20
    println(person.age)  // 출력: 20

    try {
        person.age = -5  // 예외 발생: Age cannot be negative!
    } catch (e: IllegalArgumentException) {
        println(e.message)  // 출력: Age cannot be negative!
    }
}
```

### 질문

1. 위임에 관하여 질문이 있습니다! 
    - by viewModels
    - by Delegates.observable

위임은 위 두가지로만 본적이 있는데, 보통 위임을 개발자가 구현해서 사용해야 한다면 

p.425의 CounterSet 예제처럼 위임을 사용하여 필요함 함수만 재정의 하도록 사용되는 부분이 맞을까요?.?

- 위임을 사용하지 않으면 MutableSet의 멤버 함수를 전부 재정의 해야함.

아니면, 다른 경우에도 사용된다면 어떤 느낌으로 사용될까요!?
