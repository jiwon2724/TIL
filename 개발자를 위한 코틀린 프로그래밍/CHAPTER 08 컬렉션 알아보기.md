# CHAPTER 08 컬렉션 알아보기

### 컬렉션의 가변(mutable)과 불변(immutable)

- 가변 : 컬렉션 객체의 추가, 수정, 삭제 가능
- 불변 : 컬렉션 객체 중에 한 번 만들어지면 추가, 수정, 삭제 불가능.

```kotlin
정리하자면, 불변 객체는 read-only 가변 객체는 read/write 가능임.
```

### List

- 순서가 있는 데이터의 집합임.
- 가변과 불변 객체를 생성하는 함수 `listOf`와 `mutableListOf`를 제공함.
    - `arrayListOf` 도 가변 객체를 생성하는 함수임.
- 널 값이 없는 리스트를 만들 땐 `listOfNotNull` 함수를 사용.
    - 이는 원소에 null이 있어도 제외함.
    - 내부 구현을 보면 null을 필터링 하고 있음.
        - `public fun <T : Any> listOfNotNull(vararg elements: T?): List<T> = elements.filterNotNull()`
- 아무 원소도 없는 리스트는 `emptyList` 함수로 만들 수 있음.
- 가변, 불변에 따라서 컬렉션에 원소를 추가해주는 함수의 유무가 결정됨.

<aside>
💡 XXXOf 함수들은 원소들이 없다면, 타입추론이 안됨.
</aside>

```kotlin
fun main() {
    val myList = listOf(1 ,2, 3)
    val myMutableList = mutableListOf(1, 2, 3)
    val myListOfNotNull = listOfNotNull(1, 2, 3 , null)
    
    myMutableList.add(1) // O
    myList.add(1) // X
    println(myListOfNotNull) // [1, 2, 3]
    println(myListOfNotNull[3]) // Exception in thread "main" java.lang.IndexOutOfBoundsException: Index 3 out of bounds for length 3
}
```

### Set

- 집합은 순서가 없고 모든 원소는 유일한 값을 가짐.
    - 중복 허용X
- List와 마찬가지가지로 가변과 불변이 존재하며, `XXXSetOf` 로 객체를 생성할 수 있음.
- `+=`, `-=` 연산자로 가변객체의 원소를 추가 및 삭제할 수 있음.
    - 원소의 추가 및 삭제는 함수로도 가능함.
        - `add`, `remove`
    - List도 가능.

```kotlin
fun main() {
    /** 가변 **/
    val hashSet = hashSetOf(1, 2, 3)
    val mutableSet = mutableSetOf(1, 2, 3)
    val sortedSet = sortedSetOf(1, 2, 3)
    val linkedSet = linkedSetOf(1, 2, 3)

    /** 불변 **/
    val set = setOf(1, 2, 3)

    mutableSet += 4 // add
    println(mutableSet) // [1, 2, 3, 4]
    mutableSet += 1
    println(mutableSet) // [1, 2, 3, 4]
    mutableSet -= 1 // remove
    println(mutableSet) // [2, 3, 4]
}
```

<aside>
💡 컬렉션에서 지원해주는 함수를 사용하여 각 컬렉션마다 기본 연산을 사용할 수 있음.

</aside>

### Map

- Key, Value로 구성됨.
- Key는 고유값이어야 하고, Value는 중복이 가능함.
    - Key가 중복이 되면, 가장 최근에 들어온 Key로 대체됨.
- List, Set과 마찬가지가지로 가변과 불변이 존재하며, `XXXMapOf` 로 객체를 생성할 수 있음.
    - 함수 사용 시 원소를 추가하기 위해 K, V에 대응하는 값에 `to` 혹은 `Pair`를 사용함.
- 가변 객체의 추가는 `put` 을 사용함. 삭제는 `remove`로 key값을 넣어줌.
    - `hashMap.put("ji22", "wdwd")`
    - `hashMap["ji22"] = "wdwd"` → 이렇게도 사용 가능.
- Map의 조회는 `get` 함수에 key값을 넣어 조회가 가능함.
    - 이는 `put`과 같이 대괄호로 사용이 가능함.

```kotlin
fun main() {
    val map = mapOf("hi" to "kk", "hi2" to "kkk")
    val hashMap = hashMapOf(Pair("hi", "kk"), Pair("hi2", "kkk"))
    val linkedMap = linkedMapOf("hi" to "kk", "hi2" to "kkk")

    hashMap["hi22"] = "new hi"
    println(hashMap) // {hi2=kkk, hi=kk, hi22=new hi}
    println(map.get("hi")) // map["hi"] // kk
}
```

# 컬렉션 메서드 알아보기

### 컬렉션 상속구조 알아보기

![2](https://github.com/jiwon2724/TIL/assets/70135188/1adc813b-82ab-4121-aa8e-baaa4989d5f9)


### 컬렉션 상태 확인

- 객체 생성 : 리스트(`listOf`), 집합(`setOf`), 맵(`mapOf`)
- 크기 : `size`
- 상태 : `isEmpty`
- 포함 여부 : `contains`

### 컬렉션 내부 순환

- 순환 : `forEach` → 인자로 람다를 받음.
- 인덱스가 필요한 순환 : `forEachIndexed`

### 리스트와 집합 원소 조회

- 컬렉션의 원소 접근은 대괄호나 `get` 메서드가 기본임.
- 리스트의 첫 번째 or 마지막 → `first`, `last`
- 원소 값 조회(`indexOf`, `lastIndexOf`)
    - `indexOf`는 앞에서 부터 값을 조회하고, `lastIndexOf` 는 뒤에서 부터 조회함.
    - 파라미터로 들어온 원소의 값이 없다면 -1을 반환함.
- 원소의 인덱스로 조회(`elementAt`, `elementAtOrElse` , `elementAtOrNull`)
- 특정 값 조회(`find`, `findLast` , `minOf`, `maxOf`)
    - `find` 는 람다로 들어온 조건을 앞에서 부터 조회함.
    - `findLast` 는 람다로 들어온 조건을 뒤에서 부터 조회함.
    - 만족하는 조건이 없다면 null을 반환.
    - `minOf`, `maxOf` 원소의 최소, 최댓값 반환.

### 조건 검사

- `all`, `any`, `none`으로 람다안의 조건을 검사할 수 있음.
    - `all` : 조건이 모두 참
    - `none` : 조건이 모두 거짓
    - `any` : 조건이 하나라도 만족하는 경우 참

### 정렬, 삭제, 중복처리

- 새로운 객체로 정렬 → `sorted`, `revered`
    - 이는 새로운 객체를 반환함.
- 내부 정렬 → `sort`, `reverse`
- 원소 삭제 → `drop`
    - 처음 n개 요소를 제외한 모든 요소를 포함하는 목록을 반환함.
        - n은 파라미터임.
- 여러개의 원소 조회 → `take`
    - 처음 n개 요소의 원소를 조회하는 목록을 반환함.
- 중복처리 → `distinct`

### 컬렉션 처리 함수

- `map` : 컬렉션을 순환하며 모든 원소를 변형하고 컬렉션을 반환함. 변환 로직을 람다로 받음.
- `filter` : 컬렉션을 순환하며 람다로 받은 조건이 true인 원소만 추출하여 컬렉션으로 반환함.
- `reduce` : 컬렉션을 순환하며 람다로 받은 조건을 누적하며 결과를 단일 값으로 반환함.
- `fold` : reduce와 동일하지만, 초깃값이 있음. 초깃값 부터 연산을 시작함.

```kotlin
// filter
val numbers = listOf(1, 2, 3, 4, 5, 6)
val evenNumbers = numbers.filter { it % 2 == 0 }
println(evenNumbers) // 출력: [2, 4, 6]

// map
val numbers = listOf(1, 2, 3, 4, 5, 6)
val doubledNumbers = numbers.map { it * 2 }
println(doubledNumbers) // 출력: [2, 4, 6, 8, 10, 12]

// reduce
val numbers = listOf(1, 2, 3, 4, 5, 6)
val sum = numbers.reduce { acc, i -> acc + i }
println(sum) // 출력: 21

// fold
val numbers = listOf(1, 2, 3, 4, 5, 6)
val sum = numbers.fold(10) { acc, i -> acc + i }
println(sum) // 출력: 31
```

### 컬렉션 처리 함수 연결하기 (map, filter)

```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6)
val evenSquares = numbers.filter { it % 2 == 0 }.map { it * it }
println(evenSquares) // 출력: [4, 16, 36]
```

<aside>
💡 컬렉션 처리 함수를 연쇄적으로 사용한 경우에 map 과 filter 를 어떤 순서로 해도 수행되는 문제가 있다면, 함수에 순서를 잘 조합해서 사용해야한다. 이는 연산 횟수와 관련이 있음.

</aside>

- `filter`를 먼저 사용한 경우 조건으로 걸러진 리스트에 대해 map을 수행하여 map의 연산 횟수가 줄어듦.
- `map`을 먼저 사용한 경우 연산 횟수는 줄어들지 않음.

### 그룹화

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val people = listOf(
        Person("Alice", 31),
        Person("Bob", 31),
        Person("Carol", 29),
    )
    println(people.groupBy { it.age })
}

실행결과 : {31=[Person(name=Alice, age=31), Person(name=Bob, age=31)], 29=[Person(name=Carol, age=29)]}
```

- 람다의 조건으로 그룹화를 진행함.
    - 컬렉션의 모든 원소를 어떤 특성에 따라 여러 그룹으로 나눌 수 있음.
    - 각 그룹은 리스트임. `groupBy` 의 리턴 타입은 `Map<Int, List<Person>>` 임.

### 시퀀스

- `map` 이나 `filter` 같은 함수는 컬렉션을 즉시(eagerly)생성한다.
    - 이는 컬렉션 함수를 연쇄하면, 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담음.

```kotlin
public inline fun <T> Iterable<T>.filter(predicate: (T) -> Boolean): List<T> {
    return filterTo(ArrayList<T>(), predicate)
}

public fun <T> Sequence<T>.filter(predicate: (T) -> Boolean): Sequence<T> {
    return FilteringSequence(this, true, predicate)
}
```

- `Iterable` 의 연산과 `Sequence` 의 연산은 구현이 다른걸 확인할 수 있음.
- `시퀀스(sequence)` 를 사용하면 중간 임시 컬렉션을 사용하지 않고 컬렉션 연산을 연쇄할 수 있음.
- 즉, `Iterable` 인 `filter`와 `map`으 연쇄해서 사용하는 경우는 List를 2개를 만드는 것임.

```kotlin
people.map(Person::name).filter {  }
// map의 반환과 filter의 반환으로 2개의 List가 만들어짐.
```

- `sequenceOf`를 사용하여 시퀀스 객체를 만들수 있고, `asSequence`로도 캐스팅이 가능함.

```kotlin
val list = listOf(1, 2, 3, 4, 5)
val sequence1 = list.asSequence()
val sequence2 = sequenceOf(1, 2, 3, 4, 5)
```

### 시퀀스의 중간연산과 최종 연산

- 시퀀스에 대한 연산은 **중간(`intermediate`)연산**과 **최종(`terminal`)연산**으로 나뉨.
- 중간 연산은 다른 시퀀스를 반환함.
- 최종 연산은 결과를 반환함.

```kotlin
sequence.map { ... }.filter { ... }.toList()
```

- `map`과 `filter`는 중간연산임.
- `toList` 는 최종연산임.
    - 이는 다른 컬렉션으로도 사용할 수 있음.
        - ex) `toSet`

```kotlin
fun main() {
    listOf(1, 2, 3, 4).map {
        print("map($it) ")
        it * it
    }.filter {
        print("filter($it) ")
        it % 2 == 0
    }
}

실행결과 
map(1) map(2) map(3) map(4) filter(1) filter(4) filter(9) filter(16)
```

- 시퀀스를 사용안하는 즉시 계산은 `map` 함수에 모든 원소를 람다 로직을 실행하고, 중간 결과를 새로운 컬렉션에  저장 후 `filter`  함수를 실행함.

```kotlin
fun main() {
    listOf(1, 2, 3, 4).asSequence().map {
        print("map($it) ")
        it * it
    }.filter {
        print("filter($it) ")
        it % 2 == 0
    }.toList()
}

실행결과 
map(1) filter(1) map(2) filter(4) map(3) filter(9) map(4) filter(16)
```

- 시퀀스를 사용한 경우 원소 하나씩 꺼내와 람다의 로직을 수행한다.
- 시퀀스는 최종 연산이 호출될 때 적용됨.
    - 위 예제에서 `toList` 를 빼면 아무것도 출력이 안됨.

```kotlin
listOf(1, 2, 3, 4).asSequence().map { it * it }.find { it > 3 }
```

- 시퀀스는 위 같은 로직에도 효율적임.
- 같은 연산을 시퀀스가 아닌 컬렉션에서 하게 된다면, `map`은 1, 2, 3, 4의 모든 원소에 대한 처리한 후 `find` 함수를 호출함.
- 시퀀스는 첫 번째 원소(1)에 map 함수의 람다 식을 적용 후, find 함수를 호출함.

<aside>
💡 컬렉션에 대해 수행하는 연산의 순서도 성능에 영향을 끼침.

</aside>

### 질문

1. 컬렉션 사용시 원소의 갯수가 증가할 수록 효울성이 떨어진다고 생각합니다!
    1. 컬렉션 처리 함수를 연쇄해서 사용할 경우엔 더욱
        - List가 만들어지고, 크기도 커짐.

시퀀스를 사용해서 성능개선이 가능하다고 알고있는데, 사이즈가 큰 컬렉션을 어떻게 측정해야 할까요?.?

- ex) size 1000이상 1200미만 등

2. 실무에서 시퀀스를 사용하는 빈도수가 궁금합니다! 어느정도 사이즈가 큰 컬렉션에 대해서는 연쇄적으로 컬렉션 처리 함수를 사용할 때, 시퀀스 사용을 지향하는게 좋을까요!?
