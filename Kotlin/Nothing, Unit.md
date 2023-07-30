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
