 # 코틀린에서는 모든 것이 객체이다

# 객체란?

- 항상 유일하고 특정한 자료형(클래스)를 가짐.
- 객체는 변수, 반환 값, 매개변수 등에 할당이 가능하다.
- 코틀린은 모든 것을 객체로 본다.
    - JVM으로 실행되어 자바와 동일하게 Primitive사용
    - 컴파일 처리할 때 까지 모든 객체는 해당 클래스가 있다.

### 객체

코틀린은 모든 것을 객체로 처리하기 때문에, 객체에는 해당 클래스가 있다. 다음은 클래스를 확인하는 방법이다.

- javaClass : 자바 클래스를 확인하는 속성
- javaClass.kotlin : 코틀린 클래스를 확인하는 속성

```kotlin
println("Hello".javaClass.kotlin)
println((100).javaClass.kotlin)
```

### 클래스

- 객체를 정의하는 틀 혹은 설계
- `클래스(class)` : 클래스를 정의하는 예약어, 객체를 생성하는 템플릿 도구
- `생성자(constructor)` : 객체 내의 속성들을 초기화 처리함

```kotlin
class Hello {
    val hello = "Hello"
}

val h = Hello() // 빈 생성자
```

# 객체 표현과 주석

### 리터럴, 연산자, 표현식

- 리터럴(literal) : 하나의 값. → 숫자, 문자, 문자열이다. 클래스를 만들어 객체로 만든 값은 모두 리터럴.
    - 코틀린에서 숫자, 문자, 문자열은 객체
- 연산자(operator) : 두 개의 리터럴을 연산할 때 사용하는 도구. 관례를 사용.
    - 관례 : `plus` 라는 이름의 특별한 메소드를 정의하면그 클래스의 인스턴스에 대해 `+` 연산자를 사용할 수 있음.
- 표현식(expression) : 리터럴과 연산자가 연결된 수식. 즉시 평가되어 하나의 값으로 변환.
- 객체(object) : 클래스에 의해 생성되는 모든 것

### 리터럴과 표현식 처리

숫자 100은 리터럴 표기법으로 객체이면서 값으로 사용함. 리터럴과 연산자를 연결해 표현식을 작성.

```kotlin
정수와 실수, 문자, 문자열, 논릿값도 객체이다.
정수 : Byte, Short, Int, Long
실수 : Float, Double
문자 : Char
문자열 : String
논리 : Boolean

위 타입들은 클래스임.
```

```kotlin
val intVar = 100 // default는 Int
val longVar = 100L // Long은 접미사로 L, l을 붙임
val doubleVar = 100.0 // default는 Double
val floatVar = 100.F // Float은 접미사로 F, f를 붙임.

val charVar = 'a'
val stringVar = "string"
val boolVar = true
```

### 객체는 메서드로 연산을 수행한다.

모든 객체는 상태(속성)와 행위(메서드)를 나타낸다.

- 메서드 : 클래스 내부에 정의한 함수. 객체에 `.` 을 사용하여 참조 가능.
- 함수 : 함수 이름과 호출연산자로 실행.
    - 호출 연산자 : ()
- 연산자 : 특정 기호로 표시. 관례에 따라 내부에 해당하는 메서드로 변환해 처리.

```kotlin
println(100 + 100) // 연산자 호출. -> 내부적으로 .plus() 메서드로 변환됨.
println(100.plus(100)) // 메서드 호출
```

### 주석 처리

컴파일시 실행 파일로 변경되지 않고, 프로그램 작성하는 용도로 사용.

### 한줄 주석

작성한 코드의 하나의 줄을 설명하는 용도. `//` 두 개의 슬래시로 표현.

```kotlin
val isValidToken: Boolean = false // 버튼 활성화 저장 변수
```

### 여러 줄 주석

- 코드의 설명이 길어질 땐 여러줄 주석으로 작성. `/** 내용 **/` 사이에 설명을 붙임.
- 함수나, 클래스 등 설명할 때 사용됨.

```kotlin
/**

토큰이 유효한지 검증하는 함수.
Params: token - 검증할 토큰

**/

fun isValidToken(token: String) : Boolean {
  ...
}
```

### 문자열과 문자열 템플릿

- 문자열 : 두 개의 따옴표 사이에 문자들을 순서대로 나열한 구조.
- 문자열 템플릿 : 특정 작업 진행 후 출력 시 다시 문자열로 변환하여 출력. `$` 기호를 사용함.

```kotlin
val num1 = 100
val num2 = 200 
val str = "test"

println("test : $test") // 변수가 하나인 경우
println("sum : ${num1+num2}") // 표현식을 사용한 경우
```

### Raw 문자열 처리

일반 문자나 이스케이프 문자들을 그대로 문자로 인식해서 처리하는 문자열 `"""` 를 사용함.

```kotlin
val str = """
    ㅌㅔ스트
    \n테스트
"""
ㅌㅔ스트
\n테스트
```

### 형식문자 포매팅

문자열에 특수한 편집기호를 사용. 해당 객체를 편집기호에 맞춰 처리함.

```kotlin
println("flaot = %.2f int = %d string = %s".format(3.222222, 100, "테스트"))
flaot = 3.22 int = 100 string = 테스트
```

# 값을 저장하는 변수와 상수 알아보기

### 변수

- 불변변수(`val`) : 한번 저정하면 다시 할당할 수 없음. → read-only
- 가변변수(`var`) : 재할당할 수 있는 변수 정의 → read/write

### 변수의 이름 작성

- 소문자나 언더바(`_`)로 시작.
- 첫 문자에 숫자를 쓸 수 없다.
- Camel case로 사용

### 상수

- 패키지나 `object` 키워드를 사용하는 곳에서만 정의 가능.
- `const val` 을 사용.
- 변수와 구분하기 위해 이름 모두 대문자로 사용.

```kotlin
// 변수 정의
val num1 = 100 // read-only
var num2 = 100 // read/write
num2 = 102

// 상수 정의
object Const {
    const val CONST = 100
}

println(num1)
println(Const.CONST)
```

### 타입추론과 타입변환

- 타입추론 : 변수에 초깃값을 정의한 경우 타입을 지정하지 않아도 해당 자료형으로 확정됨.
- 타입변환 : 변수에 정의된 타입이 다른 객체인 경우 `as` 를 사용하여 다른 타입으로 캐스팅 가능
    - 기본 자료형 : 숫자, 문자열 등 타입 변환을 위한`toXXX` 메서드를 제공

```kotlin
// 타입추론, 변환 예시
val num1: Int = 100 // 타입정의
var num2 = 100 // 타입추론 -> Int

val conversion = num1.toLong() // 타입변환

val list: List<String> = listOf()
val arrayList = test as ArrayList<String> // as로 타입변환
```

```kotlin
val longNum = 100L
val num = 100
val result = longNum + num // type : Long
```

정수 자료형이 다른 경우도 연산이 가능. 연산 결과는 상위 자료형으로 확정.

### 계산 연산자

- 단항연산자 : 항이 하나인 연산자. → num++
- 이항연산자 : 두 개의 항을 처리하는 연산자.  → val result = num1 + num2

### 연산자 처리 표기법

표현식은 관례에 따라서 해당 표현식에 맞는 메서드를 호출한다.

```kotlin
a + b -> a.plus(b)
a - b -> a.minus(b)

a += b -> a.plusAssign(b) // a = a + b
a -= b = a.minusAssign(b) // a = a - b

+a -> a.unaryPlus()
++a -> a.inc() 
...
```

# 질문

- 코틀린은 모든 것을 객체로 본다.
    - JVM으로 실행되어 자바와 동일하게 Primitive사용
    - 컴파일 처리할 때 까지 모든 객체는 해당 클래스가 있다.
```
JVM내에서 컴파일 시 Kotlin -> Java로 바뀌는 과정에서 코틀린의 객체들이
Primitive으로 바뀔 수 있는건 바뀌는게 맞을까요?.?  
```
```
예상 : 
val num: Int = 10 // 컴파일 시 Primitive
val user: User = User() // 컴파일 시 Reference 
```
