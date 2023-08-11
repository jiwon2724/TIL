# CHAPTER 05 : 클래스 알아보기

# 클래스 정의

- 클래스도 함수처럼 기본 구조는 머리부, 몸체부로 구분됨.
- 코틀린의 클래스는 주 생성자를 머리부에 정의함.

### 지시자 정의

지시자는 클래스의 상속이나 멤버들의 접근 범위를 제한함.

- 상속 지시자
    - `oepn` : 상속할 수 있는 클래스를 만들 때 지정
    - `final` : 코틀린 클래스의 default는 `final`임. 지시자를 표시하지 않으면 상속할 수 없는 클래스임.
- 가시성 지시자
    - 비공개(`private`) : 파일(.kt) 내부에서만 사용 가능
    - 상속만 허용(`protected`) : 파일(.kt) 내부나 상속한 경우에만 사용 가능.
    - 모듈만 허용(`internal`) : 프로젝트 내의 컴파일 단위의 모듈에서만 사용
    - 공개(`public`) : 어디서나 사용 가능. `public` 은 default임.

### 클래스 정의

```kotlin
class Person constructor(var age: Int) : Any() {
    var message = ""
    var name: String = ""
    var isMarried = false

    init {
        message = "안녕하세요"
        println(message)
    }
    
    // 주 생성자 위임 호출
    constructor(age: Int, name: String) : this(age) {
        this.name = name
        this.age = age
    }

    // 보조 생성자 위임 호출
    constructor(age: Int, name: String, isMarried: Boolean = false) : this(age, name) {
        this.name = name
        this.age = age
        this.isMarried = isMarried
    }

    fun printAge() {
        print(age)
    }
		

    class Parent() {
        // 내부 클래스 로직
    }

    object Object {
        // 내부 객체 로직
    }
}

fun main() {
    val person = Person(18) // 주 생성자 호출
    val sister = Person(19, "누나임") // 보조 생성자 호출 (주 생성자 위임 호출)
    val brother = Person(20, "형임", false) // 보조 생성자 호출 (보조 생성자 위임 호출)
}
```

- class Person(`constructor`) 주 생성자 : `class` 예약어 다음에 오는 생성자. age 프로퍼티를 생성함.
    - 주 생성자는 생략할 수 있음.

```kotlin
컴파일러는 생성자가 없는 클래스는 주 생성자를 자동으로 만들어준다.
```

- `: Any()`  : 콜론으로 클래스를 상속하거나 인터페이스를 구현할 수 있음.
- `init { ... }` : 초기화 블록임. 이는 주 생성자가 실행될 때 같이 실행됨.
    - 초기화 블록보다 위에 선언된 멤버 변수, 생성자 변수만 사용 가능.
- `constructor` : 보조 생성자. 클래스 내부에서 `constructor` 예약어를 사용해서 정의함.
    - 함수의 오버로딩처럼 여러 개 정의할 수 있음.
    - 주 생성자와 같이 정의된 경우 보조 생성자 중 하나는 주 생성자(`this`)를 호출해야함.
    - 보조 생성자가 여러개인 경우 다른 보조 생성자(`this`)를 호출해야함.
    - 보조 생성자를 정의할 땐, 주 생성자처럼 프로퍼티로 지정할 수 없음.

```kotlin
주 생성자와 보조 생성자를 모두 정의하면 항상 주 생성자까지 호출하여 전체 속성에 값을 갱신할 수 있어야 함.
너무 많은 보조 생성자가 만들어지면 불편하므로, 초깃값 등을 부여해 보조 생성자의 작성을 줄이자.
```

### 주 생성자 가시성 처리

```kotlin
fun main() {
    val singleton = Singleton.instance
}

class Singleton private constructor() {
    // 싱글톤 객체 생성
    companion object {
        val instance: Singleton by lazy { Singleton() }
    }
    // 클래스 내부 로직
}
```

- 주 생성자에 가시성을 `private` 으로 지정하면 객체를 생성할 수 없음.
    - 이 경우 `companion object` 내부에서 객체를 생성하는 메서드를 정의할 수 있음.

### 메서드 참조

- 메서드 참조는 `class`, `object` 선언 등에 지정하여 `::` 더블콜론으로 사용함.
- 객체를 생성하여 참조하는 것은 바운드, 생성하지 않고 클래스나 오브젝트를 참조하는 것은 언바운드임.
    - 언바운드는 사용시 객체를 전달해서 바운드 처리를 해야함.
- 동반객체 바운딩은 동반 객체 표기법을 사용하거나, 소괄호 안에 클래스 이름을 사용해야함.

```kotlin
fun main()  {
    println(KClass::bar) // unbound
    println(KClass()::bar) // bound
    println((KClass::bar)(KClass())) // 언바운드 바운드 처리
    println((KClass)::foo) // bound
    println(KClass.Companion::foo) // bound
}

class KClass {
    companion object {
        fun foo() {}
    }
    fun bar() {}
}

실행결과 : 
println(KClass::bar) // unbound
println(KClass()::bar) // bound
println((KClass::bar)(KClass()))
fun KClass.Companion.foo(): kotlin.Unit
fun KClass.Companion.foo(): kotlin.Unit
```

```kotlin
jetbrains.kotlin.reflect 라이브러리를 추가해야 출력결과가 잘나옴.
```

# 상속 알아보기

클래스의 계층 구조를 만드는 기법임.

- 상속관계(`is-a`) : 슈퍼 클래스와 서브 클래스가 하나로 묶여서 사용.
- 연관관계(`has-a`) : 클래스 내부에 다른 클래스를 속성으로 처리.
- 구현관계(`implements`) : 인터페이스, 추상 클래스를 구현

```kotlin
is-a 관계에만 상속시키고, has-a 관계는 컴포지션을 사용하는게 좋음.
상속은 관계가 명확하지 않을 때 사용하면, 여러가지 문제가 발생함.
즉, 단순 코드 추출, 재사용을 위해 상속을 사용하기 보단 이런 경우엔 컴포지션을 사용할 것.
is-a, has-a 관계를 생각하고 상속을 할 것인지 결정해야함.

컴포지션이란? 객체를 프로퍼티로 갖고, 함수를 호출하는 형태로 재사용 하는 것.

// 컴포지션
class Progress {
    fun showProgress() { ... }
    fun hideProgress() { ... }
}

class ImageLoader(private val progress: Progress) {
    fun load() {
        progress.showProgress()
        // 이미지 읽기
        progress.hideProgress()
    }
}
```

### 상속관계 구조화

- 클래스간 상속관계에 대한 계층이 많아질 수 있음.
    - 세 단계 이상 상속하면 사람이 이해하기 어려우므로 세 단계 정도만 상속하는 것이 좋다.

```kotlin
fun main()  {
    val student = Student("초등학교","달님", 11)
    print(student.info())
}

open class Person(val name: String) {
    open fun info() = "이름 $name"
    fun personInfo() = "재정의 불가 함수" // 재정의 불가
}

open class Man(name: String, val age: Int) : Person(name) {
    override fun info() = "이름 $name 나이 $age" // 상위 클래스 메서드 재정의
}

class Student(val school: String, name: String, age: Int) : Man(name, age) {
    override fun info() = "이름 $name 나이 $age 학교 $school" // 상위 클래스 메서드 재정의(Man)
}

// 실행결과
이름 달님 나이 11 학교 초등학교
Process finished with exit code 0
```

- 상위 클래스의 속성과 메서드가 모두 `open` 이면 하위 클래스는 이를 재정의 해야함.
    - 강제사항은 아님.
    - 하위 클래스에서 필요한 것들만 재정의할 수 있도록 제한.
        - `final` 추가
- `open` , `override` 는 하위 클래스에서 재정의 할 수 있음.

### 상속에 따른 생성자 호출

- 상속관계일 때 슈퍼클래스의 멤버를 서브 클래스에서 사용함.
    - 슈퍼클래스의 속성을 먼저 처리.
- 생성자 호출 순서도 부모 생성자가 호출되고 자식 클래스의 생성자가 호출되어야함.
- 즉 주, 보조 생성자 둘 다 위임호출이 우선순위임.

```kotlin
fun main()  {
    val pet = Pet("개", "푸들", 4)
    print("${pet.species} ${pet.subSpecies}")
}

open class Animal(val species: String) { // 슈퍼 클래스
    init {
        println("Animal")
    }
}

class Pet(species: String, val subSpecies: String) : Animal(species) { // 서브 클래스 주 생성자
    init {
        println("Pet")
    }
    constructor( // 서브 클래스 부 생성자
        species: String, 
        subSpecies: String,
        age: Int
    ) : this(species, subSpecies) // 서브 클래스 주 생성자
}

// 실행결과
Animal
Pet
개 푸들
```

# 다양한 클래스 알아보기

### 내포(중첩) 클래스

- 클래스 내부에 클래스를 정의.
    - 아무런 지시자 필요 없음.
- 클래스 내부에 정의되어 있지만, 별도의 클래스임.
- 내포 클래스를 외부에서 사용할 땐, 외부 클래스 이름으로 접근해서 사용.
- 내포 클래스에서 Outer 클래스의 멤버들을 참조할 수 없음.

```kotlin
fun main(args: Array<String>) {
    val demo = Outer.Nested()
    demo.foo()
}

class Outer {
    private val bar: Int = 1
    
    class Nested {
        private val nestVar = 100
        fun foo() = 999
    }
}
```

### 이너 클래스

- 클래스 내부에 클래스를 정의할 때 사용. `inner` 지시자를 붙여서 정의함.
- 이너 클래스의 객체는 `this` 이고, 외부 클래스의 객체는 `this@외부 클래스 이름` 으로 사용.
    - 이너 클래스 내부에서 외부 클래스의 속성에 접근 가능.

```kotlin
fun main(args: Array<String>) {
    val demo = Outer().Inner()
    println(demo.foo()) // 1
    println(Outer().getBar()) // 100
}

class Outer {
    private val bar: Int = 1

    inner class Inner {
        private val bar = 100
        fun fbar() = bar
        fun foo() = this@Outer.bar
    }
    fun getBar() = println(Inner().fbar())
}
```

### 외부 클래스의 상속관계를 이너 클래스에서 처리

상속관계를 가지는 외부 클래스를 정의하면 이너 클래스에서 속성을 참조할 때 상속관계도 지정해야함.

- `this` : 이너 클래스 객체를 참조
- `this@외부 클래스` : 이너 클래스의 외부 클래스. 즉, super 클래스를 상속한 클래스
- `super@외부 클래스` : super 클래스 참조

```kotlin
open class Super() {
    open val num = 1
}

class Outer : Super() {
    val bar: Int = 1

    inner class Inner {
        private val bar = 100
        fun fbar() = bar
        fun foo() = this@Outer.bar
        fun superFoo() = super@Outer.num // 1
    }
    fun getBar() = println(Inner().fbar())
}
```

# object 알아보기

`object` 는 클래스 정의와 하나의 객체 생성을 동시에 함 → 하나의 객체만 만들어짐 (싱글턴)

- `object` 표현식으로 익명 클래스(익명 객체)로 특정 클래스 없이 객체 생성
- `object` 선언으로 하나의 객체만 만들어서 사용
- `companion object` : 클래스에 정의해서 클래스와 같이 동반해서 사용.

### obect 표현식

```kotlin
/** 익명 클래스. 멤버들을 정의함. 이는 일회성 객체임. **/
fun getLength() : Double {
    val point = object { 
        val x: Double = 2.0
        val y: Double = 3.0
        override fun toString() = "Point($x, $y)"
    }
    return Math.sqrt(Math.pow(point.x, 2.0) + Math.pow(point.y, 2.0))
}

/** 함수 매개변수에 익명객체 전달 **/
interface Person { val name: String }
fun getObject(p: Person) = p

fun main(args: Array<String>) {
    val person = getObject(object: Person{
        override val name: String = "지원"
    })
    println("이름 ${person.name}")
}

/** 메서드의 반환값 사용 **/
interface StrType
class C {
    private fun getObject() = object : StrType { val x: String = "내부 속성의 값" }
    fun printX() { println(getObject().x) }
}

/** 클래스와 인터페이스를 상속 **/
interface Add { fun add(x: Int, z: Int) : Int }
open class A(x: Int) {
    open val y: Int = x
    open fun display() = y
}

fun main(args: Array<String>) {
    val a: Add = object: A(10), Add {
        override val y: Int = super.y
        override fun add(x: Int, z: Int) = x + y
    }
}
```

### object 정의

`object` 정의는 하나의 싱글턴 패턴을 만드는 방법임.

- `object` 정의를 처음으로 사용될 때 메모리에 로딩됨.
- 생성자가 필요 없음.
- 별도의 객체를 생성하지 않고, 정의한 이름으로 멤버들을 접근해서 사용.
- 클래스 상속, 인터페이스 구현이 가능함.

```kotlin
fun main(args: Array<String>) {
    Counter.increment()
    println(Counter.count)
}

object Counter {
    var count: Int = 0
        private set
    
    fun increment() = ++count
}
```

### 동반객체(companion object)

- 클래스와 동반 객체는 하나처럼 움직이도록 구성되어 있음.
    - 다양한 기능을 클래스 이름으로 처리 가능.
    - 동반객체는 클래스 내 하나만 생성 가능.
- 이너 클래스나, 중첩 클래스에서도 동반객체를 사용시 해당 멤버를 참조해서 사용 가능.

```kotlin
fun main() {
    CompanionClass.test() // 동반 객체 선언 : 2
    ObjectClass.ObjectTest.test() // object 선언 : 1
}

class ObjectClass {
    object ObjectTest { // 싱글턴 객체
        const val CONST_STRING = "1"
        fun test() { println("object 선언 : $CONST_STRING") }
    }
}

class CompanionClass {
    companion object { // 동반
        const val CONST_TEST = 2
        fun test() { println("동반 객체 선언 $CONST_TEST") }
    }
}
```

# 확장 알아보기

- 확장 속성 : 기존 클래스를 수정하지 않고, 새로운 속성을 추가함.
- 속성에 접근하거나 수정할 수 있는 메서드만 추가됨. → getter, setter
- `val` 로 정의시 `get` 메서드가 만들어지고, `var` 로 정의시 `get` 과 `set` 이 만들어짐.
- 정보를 관리하는 영역인 `field` 가 있음. 이를 백킹필드라고 부름.

### 최상의 속성

```kotlin
val person: Int = 0
    get() = field

var man: Int = 0
    get() = field
    set(value) { field = value }

fun main() {
    println(person)
    man = 100
    println(man)
}
```

- 최상위 속성은 `field` 를 사용할 수 있음.
    - 게터와 세터를 정의할 때 실제 값을 보관하는 필드를 사용해서 처리.
- `field` 를 사용하지 않고 게터에서 값을 반환할 수 있음.

```kotlin
val weight: Int
    get() = 100 * 2

var height: Int = 100
    get() = field * 2
    set(value) { field = value }
```

### 변경할 수 있는 속성을 변경하지 못하게 처리

- 세터를 `private` 로 처리하면 값을 변경할 수 없음.
- `private set` 을 정의하여 클래스 내부에선 변경 가능하고, 외부에서는 변경이 불가능하게 정의함.

```kotlin
class Weight(weight: Int) {
    var weight: Int = weight
        private set
    
    fun setWeight(newWeight: Int) {
        weight = newWeight
    }
}

weight = 100 // 에러
setWeight(100) // 메서드를 통해 프로퍼티를 핸들링
```

### 속성 확장

- 접두사로 리시버 클래스를 지정
    - 이는 확장을 표시함.
- 확장 속성에는 `field` 가 없음.
    - 프로퍼티를 생성하여 사용.
- `object` 나 `companion object` 도 확장이 가능함
    - `companion object` 는 `Class.Companion.이름`으로 표현

```kotlin
val String.textLength: Int
    get() = this.length // this는 리시버 클래스를 의미함.

fun main() {
    println("123123123".textLength) // 9
}
```

### 확장함수

- 확장 속성처럼 함수도 확장해서 사용이 가능함.
- 확장 속성처럼 접두사에 리시버 클래스를 지정해서 사용함.
- 마찬가지로, `object` 와 `companion object` 도 확장함수로 정의가 가능하고, 사용법은 동일함.
- 리시버의 프로퍼티를 사용할 경우엔 `this` 생략 가능함.

```kotlin
fun Context.showToast(@StringRes resId: Int) {
    Toast.makeText(this, getString(resId), Toast.LENGTH_SHORT).show()
}
```

### Nullable한 확장함수 정의

- 코틀린의 모든 클래스는 Nullable로 전환할 수 있음.
- 확장함수 내부에 null을 처리하는 코드를 추가해야함.

```kotlin
fun main() {
    val p1: Person1? = null
    p1.getName()
}

fun Person1?.getName() {
    this?.let { println(it.name) } ?: println("null")
}

class Person1(val name: String)
```

### 단일 책임 원칙(SRP)
- 모든 클래스는 하나의 책임만 가지며, 클래스는 그 책임을 완전히 캡슐화해야함.
    - 클래스가 제공하는 모든 기능은 이 책임과 부합해야함.
- 단일 책임은 어떤 클래스나 모듈, 메서드가 단 하나의 기능을 가져야 한다는 뜻.
    - 변경 사항이 발생하더라도 변경 사항에 대한 책임이 있는 부분만 수정하면 됨.

---

특정 데이터를 분석하고 서버에 전송하는 모듈을 예시로 들어보자.

- 데이터를 분석하는 알고리즘
- 서버에 전송하는 형식

이는 위 두가지 이유로 변경될 수 있음.

```kotlin
단일 책임 원칙에 의하면 두 책임을 분리된 클래스나 모듈로 나눠야함.
다른 시기에 다른 이유로 변경되어야 하는 두 가지를 묶는 것은 나쁜 설계일 수 있음.
```

한 클래스를 한 관심사에 집중하도록 유지하는 것은 중요함. 

→ 클래스를 더욱더 튼튼하게 해줌.
# 질문

p.146

- 실무와 협업에서 상속의 `is-a` , `has-a` 관계를 고려해서 프로젝트 혹은 feature를 설계하는지 궁금합니다!
    - 설계한다면 주니어 개발자의 기여도는 몇 퍼센트 정도 일까요!?

p.153 ~ p.154

- 이너 클래스와 중첩 클래스는 내부에 클래스는 포함하고 있는데, 어떤 상황에서 이 둘을 골라서 사용해야할까요!?
- 지역 클래스는 단순 일회용일까요?
    - 기능 구현시 사용하는 여부도 알고싶습니다!

p.168 ~ 169

- 동반객체와 object 정의를 클래스에 포함시킬 때, 하는 행동이 비슷해보이는데 companion object는 클래스의 이름으로만 전역적인 정보를 참조가 가능해서 사용하는걸까요~?
- 보통 사용되는 방식은 밑에 사항이 맞을까요~? class안에서 `object` 를 사용하는 경우를 못봐서 질문드립니다!
- `object` 생성(패키지에서) : 싱글턴 객체 생성
    - ex) 레트로핏
- `companion object` : 클래스의 전역적인 정보 생성
    - 모듈간 액티비티 이동시 `companion object` 로 `startActivity 호출`

p.170

- `companion object` 로 팩토리 함수를 만드는 이유가 있을까요?
    - 기본 객체 생성과 다른점이 있나요~?
