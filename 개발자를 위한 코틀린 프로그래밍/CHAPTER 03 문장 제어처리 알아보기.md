# CHAPTER : 03 문장 제어처리 알아보기

# 조건 표현식 알아보기

### 비교 연산자

- 이항연산자로 두 항의 값을 비교. 이 결과는 Boolean을 반환함.
- 동등성 비교를 위한 연산을 제외한 연산자는 관례에 따라 `compareTo` 메서드를 호출함.
- 동등성 비교는 `equlas` 메서드를 호출함.

```kotlin
val a = 10
val b = 100

println(a > b) // a.compareTo(b) > 0       false
println(a < b) // a.compareTo(b) < 0       true
println(a >= b) // a.compareTo(b) >= 0     false
println(a <= b) // a.compareTo(b) <= 0     true
```

```kotlin
compareTo() 메서드는 수신객체로 들어온 값과 파라미터로 들어온 값을 비교한다.
값이 같으면 0, 수신객체 값이 작으면 음수 -1, 크면 양수 1를 반환한다.
```

```kotlin
val a: Int? = 10
val b: Int? = 100

print(a == b) // a?.eqauls(b) ?: (b === null)       false
print(a != b) // !(a?.eqauls(b)) ?: (b === null)    true
```

```kotlin
equals() 메서드도 수신객체로 들어온 값과 파라미터로 들어온 값을 비교한다.
값(value)이 같다면 true 그렇지 않다면 false를 반환함.
수신객체의 값이 null일 경우, 파라미터로 들어온 값도 널을 참조하여 검사 후, Boolean을 반환함.
null을 명식적(==)으로 비교할 때, 코드를 최적화하는건 의미가 없다. 
== null은 === null로 자동 변환됨.
```

### 포함 연산자

- 특정 범위에 속한 값의 포함 여부를 확인할 때 사용
- 포함 여부를 결정하여 true, false를 제공
- 관례에 따라 `contains` 메서드를 호출함

```kotlin
val a = 10
val b = 0..100

println(a in b) // b.contains(a)       true
println(a !in b) // !b.contains(a)     false
```

### any, all, none

여러 원소를 가진 배열이나 리스트에 특정 원소 값이 있는지 확인하여 논리값으로 처리하는 메서드.

- `all` : 조건이 모두 참인 경우
- `none` : 조건이 모두 거짓인 경우
- `any` : 조건이 하나라도 참인 경우

```kotlin
val list = listOf(1, 2, null)

println(list.any({ it == null })) // true
println(list.all({ it == null })) // false
println(list.none({ it == null })) // false
```

### 논리연산자

- 특정 비교연산을 다시 묵어 논리값을 판단하는 연산자

```kotlin
val isAgreeTemrs = true
val isAgreePersonal = false

println(isAgreeTemrs && isAgreePersonal) // (a>b)and(a<c) false
println(isAgreeTemrs || isAgreePersonal) // (a>b)or(a<c)  true
println(!isAgreeTemrs) // false
```

```kotlin
논리합 연산은 첫 번째 항의 값에 따라서 바로 리턴될 수 있다. false인 경우 리턴.
```

### 동등성

- 객채의 값(value)이나, 참조 값(주소 값)이 같은지, 즉 동일한지 비교하는 방법.
- `==` 값(valute)을 비교 → `eqauls`
- `===` 참조(reference)를 비교

```kotlin
동일한 값의 Primitive Type은 참조연산(===)은 항상 true이다.
-> 유일하고 레퍼런스도 같음.
```

# 조건식 알아보기

- 조건식 결과를 판단해 처리하는 표현식(expression)

### if 조건

- 조건식 결과를 판단한다.
- 단일, 여러 조건을 제어할 수 있고 내부에 조건 제어 내포 가능.

```kotlin
// 한 가지 조건만 판단
if(isSuccess) startActivitiy(...)

// 두 가지 조건을 판단
if(isSuccess) startActivitiy(...) else finish()

// 표현식 처리
val userName = if (isSuccess) result.nickname else "이름없음"

// 표현식과 문장 처리 구분
val userName = if (isSuccess) {
    val newUserName = result.nickname + "#4455"
    newUserName
} else {
    showToast("이름없음")
    "이름없음"
}

// 내포된 if 조건 추가 : 
if (isSuccess) {
    ...
    if (result.data != null) {
      ...
    } else if (...)
} else {
  ...
}
```

### when 조건

- 분기가 여러개인 조건식이 있을 경우 사용
- 명령문(statement), 표현식(expression)으로 사용 가능

```kotlin
when(score) {
    00 -> print("A") // 단일 패턴 매칭
    80..89 -> print("B") // 복합 패턴 매칭
    else -> print("F") // 패턴이 없는경우(확정되지 않은 경우)
}

// 표현식
val result: String = when (score) {
    90..100 -> "A"
    80..89 -> "B"
    else -> F
}
```

```kotlin
when식의 파라미터가 확정된 값인 경우(Boolean, selead class)해당 값만 존재하기에
else를 사용 안해도됨.
```

### 예외

- 처리가 맞지 않을 경우 예외를 던질 수 있고, 컴파일러나 JVM내부 처리에서 예외를 발생 시킬 수 있다.
- 예외가 발생하면 프로그램이 중단됨.
- 이를 방지하기 위해 예외를 잡고 내부적으로 정상 처리를 해야함.

### 예외처리

- 기본적인 예외처리는 `try` , `catch` , `finally` 로 함.
- `finally` 는 옵셔널임.

```kotlin
try {
    add() // 예외가 발생할 곳에 실행
} catch (e: Exception) { // 예외가 잡히면 catch 호출
    print(e)
} finally { // 예외가 발생하거나 발생하지 않았을 때도 호출
    print("끝")
}
```

### 예외 던지기

- 예외는 `throw` 로 던지고 `catch` 로 받아 정상적인 대응을 처리해야함.

```kotlin
// 예외도 표현식이다.
val exception = try {
    throw Exception("예외 발생")
} catch (e: Exception) {
    print(e)
}
```

# 순환 표현 알아보기

### 범위

- 숫자나 문자 등 연속적인 특정 영역 이를 범위(Range)라고 함.
- 이러 범위를 특정 간격으로도 진행이 가능함.

### 범위/진행 연산자와 메서드

- 범위연산자(`..`) : 정수, 문자, 문자열 사이에 지정하여 두 항목을 포함한 범위 객체를 생성.
- `rangeTo` : 범위 연산자와 동일함.
- `until` : 범위 연산자와 동일하지만, 마지막 항목이 포함되지 않음.
- `downTo` : 역방향 범위를 만듬.
- `step` : 범위의 간격을 처리함.
- `first` , `last` , `step` : 범위의 첫 번째, 마지막, 간격 정보를 관리하는 속성

```kotlin
val range1 = 1..10 // 1 ~ 10
val range2 = 1.ranageTo(10) // 1 ~ 10
print(range1.first()) // 1
print(range2.last()) // 10

val rangeUntil = 1.until(10) // 1 ~ 9
print(rangeUntil.first()) // 1
print(rangeUntil.last()) // 9

val rangeDownTo = 10.downTo(1).step(2) // 10, 8, 6, 4, 2
print(rangeDownTo.first()) // 10
print(rangeDownTo.last()) // 2
print(rangeDownTo.step()) // 2
```

### for loop(순환)

- 동일 코드를 반복해서 처리하는 규치을 제공하는 문장.(statement)
- 범위와 진행을 사용하여 순환을 처리함.

```kotlin
// 범위 순방향
for (i in 1..5) { print("$i, ") } // 1 2 3 4 5
for (i in 1.rangeTo(5)) { print("$i ") } // 1 2 3 4 5

// 범위 간격 설정
for (i in 1..10 step 2) { print("$i ") } // 1 3 5 7 9

// 마지막 미포함
for (i in 1 until 5) { print("$i ") } // 1 2 3 4

// 범위 역방향
for (i in 5 downTo 1) { print("$i ") } // 5 4 3 2 1
```

```kotlin
중위 표기법으로 . 생략이 가능함.
```

### 순환 블록 내 코드 작성

```kotlin
for (i in 1..10) {
    if (i % 2 == 0) {
        println("continue $i")
        continue // 가장 가까운 반복문(순환)으로 이동
    }

    if (i == 7) {
        println("break $i")
        break // 가장 가까운 반복문(순환)종료(탈출)
    }
}

출력결과
continue 2
continue 4
continue 6
break 7
```

### 내포된 순환 처리(중첩 for)

```kotlin
loop@
for (i in 1..2) {
    for (j in 1..3) {
        if (j == 2) {
            println("내포 순환")
            break // 가장 가까운 for문 탈출
            // break@loop 전체순환을 종료
        }
        println("for 순환 $j")
    }
}

출력결과
for 순환 1
내포 순환
for 순환 1
내포 순환
```

### while, do while

특정 조건이 만족할 때까지 순환을 처리함. 

- `while` : 조건식을 만족할 때 까지 순환.
- `do while` : 먼저 한 번 실행 후, 조건을 확인하고 참일 경우 순환

<aside>
💡 조건식 결과가 항상 참이면 무한루프임.

</aside>

```kotlin
var n = 0
while (n < 3) { // 첫 번째 조건을 만족해야 내부 처리
    print("$n ")
    n++
} // 0 1 2

var m = 0
do { // 무조건 내부조건 한 번을 호출함.
    print("$m ")
    m++
} while (m < 3)
// 0 1 2
```

```kotlin
for문과 마찬가지로 중첩 반복문의 순환 제어문(break, continue)로 제어가 가능하고
레이블(@loop) 사용도 동일함.
```

### 반복자(Iterator)

- 범위, 배열, 리스트 등은`Iterable` 클래스임.
- 이를 상속하는 클래스는 반복할 수 있는 요소로 나타낼 수 있음.
    - 범위, 배열, 리스트가 그러함
- `iterator` 메서드로 클래스의 객체로 변환이 가능함.
- 변환한 경우 `hasNext` 와 `next` 메서드가 추가됨.
    - `hasNext` : 반복 요소가 있다면 true, 없다면 false 반환
    - `next` : 다음 요소를 반환
- 리스트나 배열처럼 인덱스를 사용할 수 없음.
- 한번 모든 원소를 조회하면 다시 객체를 생성해서 사용해야함.

```kotlin
// 외부 순환 : for, while, do while 반복문을 사용해서 처리.
val i = 1..5
val outerIterator = i.iterator()
for(j in outerIterator) { print("$j ")} // 1 2 3 4 5

// 내부 순환 : iterator 내부의 메서드를 사용하여 처리.
// outerIterator의 모든 원소를 조회했으므로, 객체를 다시 생성해야함.
val innerIterator = 1..5
innerIterator.forEach { print("$it ") } // 1 2 3 4 5

val r = 1..5.iterator()
while(r.hasNext()) { print(r.next()) } // 12345
```
