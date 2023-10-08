# String과 StringBuilder

### String

- 문자열은 연속된 문자의 배열이고, 불변(Immutable)값으로 생성되므로, 참조되는 데이터는 변경할 수 없음.
    - String 객체는 불변.
- 새로운 값을 할당할 때, 기존 메모리 외에 새로운 문자열을 위한 메모리를 만듦.

```kotlin
fun main() {
    var str = "This is String!"
    str[0] = '3' // error -> immutable
    str = "New String!" // 새로운 메모리 공간 생성
}
```

- 새로운 값이 할당 되면 기존 메모리 공간은 GC에 의해 제거됨.
    - 문자열 연산이 빈번하게 발생할 경우 매번 새로운 String 객체가 생성됨.
- 제거 시점은 개발자가 제어할 수 없기 때문에 많은 문자열을 다룰 땐 메모리 낭비에 주의해야함.
    - 대규모의 문자열을 반복적으로 조작하는 연산은 StringBuilder를 지향하는 것이 좋음.
        - 이는 메모리와 성능 측면에서 더 효율적임.

### StringBuilder

- 문자열을 조작하거나 추가할 때 사용되는 클래스임.
- 일반적인 문자열 연산에 비해 메모리 효율성과 성능상 이점을 제공함.
    - StringBuilder는 변경 가능(Mutable)한 객체임. 즉, 문자열에 변경을 가해도 새로운 객체를 생성하지 않음.
    - 문자열 연산이 빈번하게 일어날 때, String객체와 다르게 새로운 문자열 객체를 만들지 않음.
        - 이는 성능과 직결됨.

```kotlin
fun main() {
    // 생성
    val stringBuilder = StringBuilder()

    // 추가
    stringBuilder.append("This is ")
    stringBuilder.append("StringBuilder")

    // 삽입
    stringBuilder.insert(0, ">> ")

    // 삭제
    stringBuilder.delete(0, 2)
    
    // 문자열로 변환
    print(stringBuilder.toString()) // This is StringBuilder!
}
```

### 요약

- 복잡하고 빈번한 문자열 연산이 필요하다면, 성능 향상을 위해 사용을 지향하자.
    - String에 비해 메모리 및 성능 효율성 뛰어남.
