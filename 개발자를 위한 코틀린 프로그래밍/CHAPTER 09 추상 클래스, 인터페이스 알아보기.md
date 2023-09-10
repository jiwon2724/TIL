# CHAPTER 09 : 추상 클래스, 인터페이스 알아보기

# 추상 클래스 알아보기

### 추상 클래스

- 추상 클래스는 `abstract` 예약어로 지정한 클래스임.
- 추상 클래스는 직접 객체를 생성할 수 없고, 다른 클래스에 상속해서 구현해야함.
- 추상 클래스에서는 추상 속성, 메서드 이외의 일반 속성과 일반 메서드도 정의할 수 있음.
- 추상 클래스는 전체적인 구성이 “구체화” 되어 있지 않고, 설계만 되어 있는 클래스임.
- 여러 클래스에 걸쳐 공통적으로 사용되는 메서드나 변수를 추상 클래스에 정의함으로써 코드 중복을 줄일 수 있음.

```kotlin
fun main() {
    val man = Man("지원").apply {
        println(name)
        displayJob("안드로이드 개발자")
        println(age)
        displaySSN(1234)
    }
}

class Man(override val name: String) : Person(name) {
    override fun displayJob(description: String) { println("$name 님의 직업은 $description 입니다.") }
    override fun displaySSN(ssn: Int) { super.displaySSN(ssn) } // 재정의 필수 아님.
}

abstract class Person(name: String) {
    init { println("이름은 $name") }

    var age: Int = 40  // 일반 속성
    abstract val name: String // 추상 속성

    open fun displaySSN(ssn: Int) { println("주민번호 is : $ssn") } // 일반 메서드
    abstract fun displayJob(description: String) // 추상 메서드
}

이름은 지원
지원
지원 님의 직업은 안드로이드 개발자 입니다.
40
주민번호 is : 1234
```

- `abstract` 예약어를 붙인 추상 속성(멤버)은 반드시 구현 클래스에서 재정의 되어야 함.
- `open` 예약어를 붙인 멤버는 구현 클래스에서 재정의 될 수 있음.
- `init` 블록으로 주 생성자의 특정 기능을 처리할 초기화 블록을 제공함.

### 추상 클래스 활용

- 추상 클래스를 정의하면 여러 클래스에 동일한 규칙을 제한할 수 있는 공통 기능를 제공함.
    - ex) 안드로이드의 `BaseXXX`

```kotlin
abstract class BaseActivity<T : ViewDataBinding>(@LayoutRes private val layoutId: Int) : AppCompatActivity() {
    protected val binding: T by lazy(LazyThreadSafetyMode.NONE) {
        DataBindingUtil.setContentView(this, layoutId)
    }

    init {
        addOnContextAvailableListener {
            binding.notifyChange()
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding.lifecycleOwner = this
    }

    override fun onDestroy() {
        binding.unbind()
        super.onDestroy()
    }

    protected inline fun bind(block: T.() -> Unit) {
        binding.apply(block)
    }
}
```

- 위 `BaseActivity` 코드는 데이터바인딩을 초기화 해주는 보일러 플레이트 코드를 해소시키고, 동일한 규칙을 제공함.

# 인터페이스 알아보기

### 클래스와 인터페이스의 차이

- 클래스는 객체를 생성해 내부의 속성과 메서드를 사용함.
- 인터페이스는 추상 속성과, 추상 메서드를 기본으로 처리함. 이는 추상 클래스처럼 객체를 생성할 수 없음.

### 상속과 구현의 차이

- 상속은 추상 클래스 or 일반 클래스 간의 관계를 구성함.
- 구현은 클래스가 인터페이스 내의 추상 멤버를 구현하는 것.

### 인터페이스

- `interface` 예약어를 사용해서 정의해야함.
- 다중 구현으로 추상 멤버들을 재정의할 수 있음.
    - 코틀린은 단일 상속만 가능함.
- '무엇'을 해야 하는지에 대해 설명하고 '어떻게' 해야 하는지는 구현하지 않음.
    - 구현하는 클래스에서 행위를 작성함.
- 인터페이스를 통해 다양한 객체를 동일한 참조 변수로 다룰 수 있음.
    - 다형성

```kotlin
fun main() {
    with(TvVolume()) {
        up()
        up()
        up()
        down()
        showVolume()
    }
}

class TvVolume(override var currentVolume: Int = 0) : Clickable {
    override fun up() {
        println("볼륩 올리기")
        currentVolume++
    }
    override fun down() {
        println("볼륩 내리기")
        currentVolume--
    }

    override fun showVolume() {
        println("현재 볼륨은 : $currentVolume")
    }
}

interface Clickable {
    var currentVolume: Int
    fun up()
    fun down()
    fun showVolume()
}

실행결과
볼륩 올리기
볼륩 올리기
볼륩 올리기
볼륩 내리기
현재 볼륨은 : 2
```

- 인터페이스에도 일반 속성과 일반 메서드를 정의할 수 있음.
- 인터페이스의 프로퍼티는 커스텀 게터, 세터를 지원하지 않음.
    - `val` 인경우 커스텀 게터만 지원함.

### 여러 인터페이스 상속 구현

- 여러개의 인터페이스를 구현하는 경우엔 각 인터페이스에 있는 추상 속성과, 추상 메서드를 모두 구현해야함.

```kotlin
fun main() {
    val duck = Duck()
    duck.fly()
    duck.swim()
}

class Duck : Flyer, Swimmer

interface Flyer {
    fun fly() { println("오리는 날수 있어요") }
}

interface Swimmer {
    fun swim() { println("수영도 가능함!") }
}

실행결과
오리는 날수 있어요
수영도 가능함!
```

### 인터페이스 계층 처리

- 계층을 만드는 이유는 구현 클래스가 다양할 경우 구현 클래스의 상속을 계층에 맞춰서 만들 수 있음.
    - ex) 컬렉션 프레임 워크를 보면 대부분 인터페이스로 계층을 만들고, 필요한 경우 구현 클래스가 상속받아 인터페이스 범위에 따라 작동하도록 되어있음.

```kotlin
fun main() {
    val myLaptop = Alienware()
    myLaptop.compute()
    myLaptop.chargeBattery()
    myLaptop.activateTurboMode()
}

class Alienware : GamingLaptop {
    override fun compute() { println("게이밍 노트북은 매우 빠르죠?") }
    override fun chargeBattery() { println("게이밍 노트북 충전중이에요!") }
    override fun activateTurboMode() { println("터보 모드 가동중!") }
}

interface Computer {
    fun compute()
}

interface Laptop : Computer {
    fun chargeBattery()
}

interface GamingLaptop : Laptop {
    fun activateTurboMode()
}

실행결과
게이밍 노트북은 매우 빠르죠?
게이밍 노트북 충전중이에요!
터보 모드 가동중!
```

# 봉인(sealed) 클래스 알아보기

- 코틀린 컴파일러는 when 사용시 디폴트 분기인 else를 덧붙이게 강제함.
    - `sealed` 클래스로 이를 해결할 수 있음.
- `sealed` 예약어를 지정하여 클래스를 정의함.
- 상위 클래스에 `sealed` 예약어를 붙이면 해당 상위 클래스를 상속한 하위 클래스의 정의를 제한할 수 있음.
    - 봉인함.
- 서브 클래스는 `class`, `data class`, `object` , `data object` 모두 가능함.

```kotlin
sealed class UiState<out T> {
    object Loading: UiState<Nothing>()
    data class Success<T>(val data: T): UiState<T>()
    data class Error(val error: Throwable?): UiState<Nothing>()
}

// 처리
when (uiState) {
    is UiState.Loading -> TODO("")
    is UiState.Error -> TODO("")
    is UiState.Success -> TODO("")
}
```

- uiState의 하위 클래스가 정해져 있음. 즉, UiState 클래스에 Loading, Success, Error 이외에 다른 클래스는 없음.
    - uiState의 하위 클래스를 모두 적용하면 else 강제화 X
    - 그렇지 않다면 else를 사용해야함.
- 안드로이드에서 상태관리에 많이 사용됨.

```kotlin
sealed class Menu(val name: String, val price: Int){
    class Americano(name:String, price:Int, val wondu: String): Menu(name, price)
    class Latte(name:String, price:Int, val milk : String): Menu(name, price)
    class MSGR(name:String, price:Int, val dangdo: String): Menu(name, price)
}

fun main() {
    val order1 = Menu.Americano("아이스 아메리카노", 5000, "콜롬비아")
    val order2 = Menu.Latte("라떼", 6500, "저지방 우유")
    val order3 = Menu.MSGR("미숫가루", 6000, "매우 달게")
    
    val myOrder: Menu = order3
    print("주문하신 음료는 ${myOrder.name}입니다. 가격은 ${myOrder.price}이고 ")
    when(myOrder){
        is Menu.Americano -> print("원두는 ${myOrder.wondu}입니다.")
        is Menu.Latte -> print("우유는 ${myOrder.milk}입니다.")
        is Menu.MSGR -> print("당도는 ${myOrder.dangdo}입니다.") 
    }
}
주문하신 음료는 미숫가루입니다. 가격은 6000이고 당도는 매우 달게입니다.
```

### 질문

- 인터페이스에서 프로퍼티가 사용되는 경우가 있을까요?.? 행위를 구현하다보니 프로퍼티를 사용하는 경우는 못봐서 질문드립니다!
    - 제 생각은 프로퍼티는 상태를 나타내므로, 인터페이스 보단 인터페이스를 구현하는 클래스 안에서 관리하는게 나아서 인터페이스 안에서 프로퍼티 사용을 못봤던 것 같아요!
- 추상 클래스와 인터페이스를 사용되는 경우는 다음과 같은게 맞을까요~?
    - 추상 클래스 : 공통된 기본 기능을 구현할 때, 상속을 통해 코드 재사용이 필요할 때.
        - BaseXXX
    - 인터페이스 : 행위(event, action)을 정의할 때.
        - 클릭, 스와이프 등등

### 멘토님 답변
- sealed class는 추상 클래스임.
  - 클래스간 범위를 제안함. 무분별하게 상속을 막아줌.
- private class는 사용하는 경우는 겨우 없음.
- 인터페이스에서 프로퍼티가 쓰임. ex) tv 볼륨의 상태 게터와 세터로
- 공통된 기본 기능을 구현할 때 -> 추상 클래스 : 구현체들의 추상적인 개념 존재하진 않지만 개념적으로 알고있는.
- enum, sealed class시 when식에서 else를 생략하는건 좋은 것임.
  - 새로 추가해야하는 부분에서 작성을 안할경우 컴파일 에러.
  - 이러한 부분에서 개발자가 까먹지 않게 도와주는 부분이 있음.
  - 즉, when을 사용하여 이러한 부분들을 잡아줄 수 있음.

