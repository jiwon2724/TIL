# BFS

- 너비 우선 탐색(Breadth-First Search)그래프 탐색 알고리즘임.
- 하나의 정점으로부터 시작해 차례때로 모든 정점들을 한 번씩 방문하는 것.
    - 정점은 `Node` or `Vertex`
- Queue로 구현 가능.

### 알고리즘
![images_lucky-korma_post_2112183b-bfcd-427e-8072-c9dc983180ba_R1280x0-2](https://github.com/jiwon2724/TIL/assets/70135188/5e576908-b0e3-4f24-bc2b-8b3ee24f9b89)


- 루트노드 or 임의의 노드에서 시작함.
- 인접한 노드를 먼저 탐색하는 방법으로, 시작 정점으로부터 가까운 정점을 먼저 방문하고 멀리 떨어져 있는 정점을 나중에 방문하는 방법임.
    - 즉, 지금 위치에서 갈 수 있는 것들을 체크하는 것.
        - 큐에 넣는 방식.
    - 1의 인접 노드 방문 → 2, 3, 4
    - 2, 3, 4의 인접 노드 방문 → 5, 6, 7, 8
    - 위 과정 반복.

### 소스코드 - Kotlin

```kotlin
fun main() {
    // 노드가 연결된 간선 -> 인접 리스트
    val graph = mapOf(
        1 to listOf(2, 3, 4),
        2 to listOf(1, 4),
        3 to listOf(1, 4),
        4 to listOf(1, 2, 3)
    )

    println(bfs(graph, 1)) // 1, 2, 3, 4
}

fun bfs(graph: Map<Int, List<Int>>, start: Int): List<Int> {
    val visited = mutableSetOf<Int>()
    val queue: Queue<Int> = LinkedList<Int>()
    val result = mutableListOf<Int>()

    queue.add(start)

    while (queue.isNotEmpty()) {
        val current = queue.poll()
        if (current !in visited) {
            visited.add(current)
            result.add(current)
            graph[current]?.forEach {
                if (it !in visited) {
                    queue.add(it)
                }
            }
        }
    }
    return result
}
```

### 범위 및 함수 사용시 유의사항

- `indices` : `IntRange`타입을 반환하며**,** 수신객체의 전체 범위 반환. 위 예제 에선 0..6
- `until` : 범위의 마지막 요소를 포함하지 않음.
    - `0..5` => 0, 1, 2, 3, 4, 5
    - `0 until 5` => 0, 1, 2, 3, 4
- `numbers.size` , `numbers.lastIndex` 을 주의해서 사용.

### 장점

- 최단 경로를 보장함.
    - 인접 노드를 우선적으로 탐색.

### 단점

- 탐색 공간이 크고, 목표 노드가 깊은 레벨에 있다면 비효율적일 수도 있음.

### 시간 복잡도

- O(V+E) → 인접 리스트인 경우
    - 간선 + 노드
