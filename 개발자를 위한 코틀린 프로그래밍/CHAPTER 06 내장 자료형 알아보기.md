# 문자와 문자열 자료형

- 문자열은 기본으로 변경할 수 없는 객체임.
    - 즉 내부 원소를 추가하거나, 변경이 불가능함.
    - 문자열을 변경하려면 변경 가능한 `StringBuilder` 클래스를 사용해야함.

```kotlin
var a: String = "abc"
a = "def"
```

- 위 코드는 a 변수가 참조하고 있는 공간에 “def”로 변경한게 아님.
- “def”라는 새로운 인스턴스를 생성해서 a가 해당 인스턴스를 참조하도록 만듦.
    - 이전에 참조된 “abc”는 가비지 컬렉터에 의해 처리됨.
- 문자열은 문자를 원소로 갖는다. 즉, 내부의 원소를 조회할 수 있음.
    - 문자열 메서드로 내부를 조회하는 메서드는 다음과 같음.
        - `get`, `first`, `last`, `legnth`, 등

### 빈 문자열 처리

- `isEmpty` : 빈 문자열 체크, 문자열의 길이가 0인 경우에만 true를 반환함.
- `isBlank` : 빈 문자열 체크, 공백 문자만 포함한다면 true를 반환함.
    - 공백 문자는 space(공백), \t(탭), \n(줄 바꿈)등 을 의미함.
- `trimEnd` : 마지막 공백만 제거
- `trimStart` : 처음 공백만 제거
- `trim` : 앞, 뒤 공백 제거
    - 공백 문자는 space(공백), \t(탭), \n(줄 바꿈)등 을 의미함.

### 문자열 비교와 대소문자 등의 메서드

- `replaceFirstChar` : 문자열의 첫 글자를 대, 소문자로 변경

```kotlin
val str = "eagle"
str.replaceFirstChar { it.uppercase() } // Eagle
```

- `uppercase/lowercase` : 문자열을 대, 소문자로 변경

```kotlin
val str = "eagle"
val upperStr = str.uppercase()
println(upperStr) // EAGLE
println(upperStr.lowercase()) // eagle
```

### 문자열 매치 메서드

- `startsWith/endsWith` : 첫, 마지막 글자의 파라미터의 문자열을 매치함.

```kotlin
val str = "eagle"
println(str.startsWith("e")) // true
```

- `find/findLast` : 주어진 조건에 일치하는 첫 번째, 마지막 원소를 반환함.
    - 일치하는 원소가 없다면 null 반환.

```kotlin
val text = "Hello, World!"

val firstUpper = text.find { it.isUpperCase() }
println(firstUpper) // H 

val lastUpper = text.findLast { it.isUpperCase() }
println(lastUpper)  // W
```

### 변경 가능한 문자열 StringBuilder 처리

- `append` : 마지막 인덱스 뒤에 추가
- `insert` : 특정 인덱스 위치에 추가
- `clear` : 전체 삭제
- 등

# Any, Unit, Nothing 클래스

- `Any`
    - 클래스 정의 시 아무것도 상속하지 않아도 `Any` 클래스를 자동으로 상속함.
        - 모든 클래스의 최상위 클래스는 `Any`임.
- `Unit`
    - 함수는 항상 반환값을 처리함. 반환값이 없다는 것은 `Unit`의 객체를 반환하는 것.
    - `Unit` 도 `Any`를 상속하고 있음.
- `Nothing`
    - 반환 시 아무것도 없다는 것을 `Nothing`으로 표현함.
    - 무한 루프나 `throw`등을 포함하는 함수는 실제로 값을 반환하지 않으므로 `Nothing` 타입을 반환 타입으로 가질 수 있음.
        - 예외를 발생할 때 사용함.
    
    ```kotlin
    fun fail(message: String): Nothing {
        throw IllegalArgumentException(message)
    }
    ```
    
    위의 `fail` 함수는 항상 예외를 던지기 때문에 반환 값이 없음. 즉, 반환 자체를 하지 않음.

  # Unit
- **The type with only one value: the Unit object. This type corresponds to the void type in Java.**
- 코틀린 문서에 따르면 `Unit`에 대한 설명을 위 처럼 해줌. 이는 Java의 void 역할이고, 이는 객체임.
    - 즉, 코틀린의 모든 객체는 Any의 자식이므로 Unit은 Any의 서브 클래스임.
```kotlin
    // Kotlin
    fun returnTest1(): Unit { }
    fun returnTest2(): Unit { return }
    fun returnTest3(): Unit { return Unit } // fun returnTest3(): Unit = Unit

    // Java - Decompile
    public static final void returnTest1() { }
    public static final void returnTest2() { }
    public static final void returnTest3() { }
```
- 명시적으로 return 값을 작성하지 않아도 됨.
- void로 디컴파일됨.
# Nothing
- **Nothing has no instances. You can use Nothing to represent "a value that never exists": for example, if a function has the return type of Nothing, it means that it never returns (always throws an exception).**
- 코틀린 문서에 따르면 `Nothing`은 인스턴스를 생성할 수 없음.
```kotlin
public final class Nothing private constructor() {}
```
- `Nothing`은 디컴파일 시 `Void`(대문자)가 나오는데, 이는 인스턴스화 할 수 없는 `void`를 나타냄.
- 값이 없는경우(어떤 값도 얻을 수 없음) `Nothing`을 사용할 수 있고, 함수의 반환 타입이 `Nothing`이면 반환하지 않는다는 의미임.
  - 즉, 리턴 될 일이 없을 경우 사용
```kotlin
Unit은 리턴의 대상이 없을 뿐 반환을 하지만, Nothing은 리턴이라는 행위 자체를 하지 않음.
```
다시 돌아와서 코틀린 문서에는 다음과 같은 글이 있이 있음.

The throw expression has the type Nothing. This type has no values and is used to mark code locations that can never be reached. In your own code, you can use Nothing to mark a function that never returns:

```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```
- `throw`는 `Nothing`타입을 가짐. (함수가 리턴될 일이 없는 경우 위 상황과도 맞아 떨어짐.)
  - 예외를 던지는 리턴타입
- 이는 값이 없으며, `throw` 이후 코드는 실행되지 않음. (used to mark code locations that can never be reached.)
- `throw`는 표현식임. 다음 예제로 `Nothing`이 어떻게 캐스팅되는지 확인해보자.
```kotlin
  val s: String? = null
  val result = s ?: throw IllegalArgumentException("Name required")
```
- 위 코드는 코틀린 문서에 나오는 예제를 변형한 것임.
- 예상대로라면, 코틀린 컴파일러가 해당 부분에 힌트를 줄거라 생각하지만 그렇지 않음.
- 그 이유는 `Nothing`타입은 모든 타입의 서브 타입이라 항상 `String` 타입으로 평가됨.

# 정리
- `Unit`과 `Nothing`은 사용하는 용도에 따라서 다름.
  - `Nothing` : `throw`를 사용하여 예외를 던지는 경우나, 리턴될 경우가 없는 상황
  - `Unit` : 반환이라는 행위를 하되, 아무값도 반환하지 않는 상황
  - 예외시 `Unit`을 사용하지 않는 경우는 `Unit`을 반환하는 함수의 코드블록은 예외를 만나고 다음 블록을 실행할 것임.
    - 즉 상황에 맞게 `Unit`과 `Nothing`을 적절하게 사용하는 것이 중요함.

    
    # 자료형 처리 알아보기
    
    ### Nullable 여부
    
    - 기본 자료형에 `?` 를 붙이면 Nullable한 객체임.
        - ex) `String?`, `Int?`, `Person?`
    
    ### Nullable 자료형 점검 후 처리
    
    ```kotlin
    fun main() {
        val str: String? = "Kotlin"
        var strLength: Int? = 0
        
        if(str != null) { strLength = str.length } // null이 아닌경우 처리
        strLength = str?.length // 위 코드와 동일한 식임.
    }
    ```
    
    ### Nullable 처리 연산자 자세히 알아보기
    
    - 단언 연산자(`!!`) : 널이 들어오면 안되는 곳에 연산자 처리 전에 지정함.
    - 안전(safe call) 연산자(`?.`) : 널이 들어오면 그 다음 메서드를 처리하지 않음.
    - 엘비스(Elvis) 연산자(`?:`) : safe call과 같이 사용해서 널이 발생한 경우 별도의 값으로 처리할 수 있음.
    
    ```kotlin
    val str: String? = null
    var strLength: Int? = 0
    
    if(str != null) { strLength = str.length }  // strLength = 0
    strLength = str?.length // strLength = null
    
    var length = str?.length ?: 0 // 0
    length = str!!.length // NPE
    ```
