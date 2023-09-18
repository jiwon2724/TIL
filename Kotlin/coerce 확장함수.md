# coerce 확장함수
- 특정 범위나 한계에 값을 제한하는 유용한 확장함수임.
- `coerceIn`, `coerceAtLeast`, `coerceAtMost`가 있음.

### coerceIn

- 수신객체 값이 주어진 범위 내에 있지 않으면 가장 근접한(최소, 최댓값)을 반환하고 그렇지 않으면(범위 내에 있으면) 원래 값을 반환함.
- 파라미터로 범위(range)를 지정해서 사용 가능함.

```kotlin
public fun <T : Comparable<T>> T.coerceIn(minimumValue: T?, maximumValue: T?): T {
    if (minimumValue !== null && maximumValue !== null) {
        if (minimumValue > maximumValue) throw IllegalArgumentException("Cannot coerce value to an empty range: maximum $maximumValue is less than minimum $minimumValue.")
        if (this < minimumValue) return minimumValue
        if (this > maximumValue) return maximumValue
    }
    else {
        if (minimumValue !== null && this < minimumValue) return minimumValue
        if (maximumValue !== null && this > maximumValue) return maximumValue
    }
    return this
}
```

```kotlin
fun main() {
		println(5.coerceIn(0, 10)) // 범위내에 있음. 기본값 반환 : 5
    println((-1).coerceIn(0, 50)) // 범위내에 없음. 최솟값 반환 : 0
    println(1000.coerceIn(0, 50)) // 범위내에 없음. 최댓값 반환 : 50
    println(1000.coerceIn(1..999)) // ..으로 범위연산 사용가능 최댓값 반환 : 999
}
```

### coerceAtLeast

- 주어진 값보다 크거나 같은 값을 보장하기 위해 사용.
    - 수신객체 값이 주어진 최솟값보다 크거나 같다면 기본값을 반환.
    - 그렇지 않다면 주어진 최솟값을 반환함.

```kotlin
public fun <T : Comparable<T>> T.coerceAtLeast(minimumValue: T): T {
    return if (this < minimumValue) minimumValue else this
}
```

```kotlin
fun main() {
    println(5.coerceAtLeast(3)) // 최솟값 보다 기본값이 더 큼 : 5
    println(1.coerceAtLeast(3)) // 최솟값 보다 기본값이 더 작음 : 3
}
```

### coerceAtMost

- 주어진 값보다 작거나 같은 값을 보장하기 위해 사용.
    - 수신객체 값이 주어진 최댓값보다 작거나 같다면 기본값을 반환.
    - 그렇지 않다면 주어진 최댓값을 반환함.

```kotlin
public fun <T : Comparable<T>> T.coerceAtMost(maximumValue: T): T {
    return if (this > maximumValue) maximumValue else this
}
```

```kotlin
fun main() {
    println(5.coerceAtMost(3)) // 최댓값 보다 기본값이 더 큼 : 3
    println(1.coerceAtMost(3)) // 최댓값 보다 기본값이 더 작음 : 1
}
```
