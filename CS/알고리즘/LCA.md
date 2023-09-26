# LCA

- LCA(Lowest Common Ancestor) 최소 공통 조상이라 하며, 두 노드의 공통된 조상 중에서 가장 가까운 조상을 찾는 알고리즘임.

### 알고리즘

- 모든 노드에 대한 깊이(depth)를 계산함.
    - dfs로 트리의 depth를 구할 수 있음.
- 최소 공통 조상을 찾을 두 노드를 확인함.
    1. 먼저 두 노드의 깊이가 동일하도록 거슬러 올라감.
    2. 그 이후 부모가 같아질 때까지 반복적으로 두 노드의 부모 방향으로 거슬러 올라감.
    3. 모든 연산(LCA(a, b))에 대해서 2번 과정을 반복.

### 소스코드 - Kotlin

```kotlin
// 트리의 depth 구하기
fun dfs(curr: Int, d: Int) {
    depth[curr] = d
    for (next in graph[curr]) {
        if (depth[next] == 0) {
            parent[next] = curr
            dfs(next, d + 1)
        }
    }
}

// 최소 공통 조상 찾기
fun lca(a: Int, b: Int): Int {
    var aa = a
    var bb = b
    // 깊이 조정
    while (depth[aa] != depth[bb]) {
        if (depth[aa] > depth[bb]) aa = parent[aa]
        else bb = parent[bb]
    }
    // 노드가 같아지도록
    while (aa != bb) {
        aa = parent[aa]
        bb = parent[bb]
    }

    return aa
}
```

### 범위 및 함수 사용시 유의사항

- `indices` : `IntRange`타입을 반환하며**,** 수신객체의 전체 범위 반환. 위 예제 에선 0..6
- `until` : 범위의 마지막 요소를 포함하지 않음.
    - `0..5` => 0, 1, 2, 3, 4, 5
    - `0 until 5` => 0, 1, 2, 3, 4
- `numbers.size` , `numbers.lastIndex` 을 주의해서 사용.

### 시간 복잡도

- O(N)
    - 모든 쿼리를 처리할 땐 O(N*M)
