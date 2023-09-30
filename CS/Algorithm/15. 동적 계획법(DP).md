# DP
- DP(Dynamic Programming) 동적 계획법이라고 함.
- 복잡한 문제를 더 작은 하위 문제로 분해하여 각 하위 문제의 해결책을 이용해 문제를 해결하는 최적화 기법.
- 한 가지 문제에 대해서, 단 한 번만 풀도록 만들어주는 알고리즘임.
    - 메모이제이션 : 이미 계산된 문제의 해결책을 메모리에 저장.
    - 똑같은 연산을 반복하지 않도록 만들어줌.
        - ex) 답을 구하기 위해 이미 했던 계산을 반복하는 문제일 경우

### 알고리즘

- 작은 문제에서 반복이 일어나는 경우와 같은 문제에 대한 정답이 같은 경우 배열에 값들을 저장함.
- 그 이후 같은 문제에 대해 저장된 값이 있다면 연산을 하지말고 저장된 배열의 값을 반환.

### 소스코드 - Kotlin

```kotlin
lateinit var dp: Array<Int>

fun main() {
    val n = readIn().toInt()
    dp = Array(n+1) { 0 }
    val answer = fibo(n)
    println(answer)
}

fun fibo(n: Int): Int {
    if(n == 0) return 0
    if(n == 1 || n == 2) return 1
    if(dp[n] != 0) return dp[n] // 같은 연산에 대한 값이 저장되어있는지 확인
    dp[n] = (fibo(n-1)  + fibo(n-2)) // 값 저장
    return dp[n]
}
```

### 범위 및 함수 사용시 유의사항

- `indices` : `IntRange`타입을 반환하며**,** 수신객체의 전체 범위 반환. 위 예제 에선 0..6
- `until` : 범위의 마지막 요소를 포함하지 않음.
    - `0..5` => 0, 1, 2, 3, 4, 5
    - `0 until 5` => 0, 1, 2, 3, 4
- `numbers.size` , `numbers.lastIndex` 을 주의해서 사용.

### 시간 복잡도

- 문제 상태에 따라 달라짐.
- 위 예제인 피보나치수열 같은 경우는 O(2^N)을 O(N)으로 성능을 높일 수 있음.
