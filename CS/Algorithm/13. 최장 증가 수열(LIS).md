# LIS

- 최장 증가 수열(Longest Increasing Subsequence)은 어떤 수열에서 일부 숫자를 지워 만들 수 있는 증가하는 부분 수열중 가장 긴 것을 찾는 알고리즘임.
- DP와 이분 탐색을 활용하여 문제를 접근함.
    - DP(Dynamic Programming)는 동적 계획법으로 분할 정복과 비슷하지만, 작은 문제에서 중복이 일어나는지 체크함.
    - 정답을 구현한 문제를 배열에 저장하여 같은 문제가 나온다면 배열에 저장한 문제의 결과값을 이용.

### 알고리즘

- `[ 6, 2, 5, 1, 7, 4, 8, 3]` 배열중 LIS는 `2, 5, 7, 8`임.
- 배열 사이즈 만큼 값을 저장할 배열을 1로 초기화 함.
    - 원소 자체만으로 1의 길이를 가진 증가하는 부분 수열을 형성할 수 있기 때문임.
- j가 i만큼 순회를 시작함.
    - arr[i] > arr[j] 일 때 arr[i]가 더 크다면 부분 수열에 arr[j]다음 arr[i]가 올 수 있는 것임.
    - i가 2인경우
        - 5 > 6 수열X
        - 5 > 2 수열O → 2, 5로 수열이 가능하므로 dp의 값을 증가시키는 것임.
            - 즉 2번째 인덱스(5)의 dp값은 2임. 수열의 길이가 2라는 뜻.
- 위 순회를 반복하여 가장 값(수열의 길이)이 큰 dp배열의 값을 리턴함.

### 소스코드 - Kotlin

```kotlin
fun lisDP(arr: IntArray): Int {
    val n = arr.size
    val dp = IntArray(n) { 1 } 

    for (i in 1 until n) {
        for (j in 0 until i) {
            if (arr[i] > arr[j] && dp[i] < dp[j] + 1) {
                dp[i] = dp[j] + 1
            }
        }
    }
    return dp.maxOrNull() ?: 0
}
```

### 범위 및 함수 사용시 유의사항

- `indices` : `IntRange`타입을 반환하며, 수신객체의 전체 범위 반환. 위 예제 에선 0..6
- `until` : 범위의 마지막 요소를 포함하지 않음.
    - `0..5` => 0, 1, 2, 3, 4, 5
    - `0 until 5` => 0, 1, 2, 3, 4
- `numbers.size` , `numbers.lastIndex` 을 주의해서 사용.

### 시간 복잡도

- DP의 경우 : O(n^2)
- 이분탐색 : O(log n)
