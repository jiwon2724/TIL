# Dijkstra

- 다익스트라(Dijkstra)는 최단 경로 탐색 알고리즘임.
    - 가장 비용이 적은 경로
- 특정한 하나의 정점(Vertex of Node)에서 다른 모든 정점으로 가는 최단 경로를 알려줌.
    - 간선의 비용이 음수일 땐 포함X
- 하나의 최단 거리를 구할 때 그 이전까지 구했던 최단 거리 정보를 그대로 사용함.
- 현재까지 알고 있던 최단 경로를 계속해서 갱신함.
- 우선 순위 큐를 사용하여 구현함.

### 알고리즘

1. 출발 노드(정점) 선택
2. 출발 노드를 기준으로 각 노드의 최소 비용 저장
3. 방문하지 않은 노드 중 가장 비용이 적은 노드 선택
4. 해당 노드를 거쳐 특정한 노드로 가는 경우를 고려하여 최소 비용 갱신
5. 3 ~ 4번 반복

### 소스코드 - Kotlin

```kotlin
data class Edge(val to: Int, val weight: Int)

fun dijkstra(start: Int, graph: List<List<Edge>>, numVertices: Int): IntArray {
    val distances = IntArray(numVertices) { Int.MAX_VALUE }
    distances[start] = 0

    val pq = PriorityQueue<Pair<Int, Int>>(compareBy { it.second }) // <vertex, distance>
    pq.offer(Pair(start, 0))

    // 최단 거리 탐색
    while (pq.isNotEmpty()) { 
        // 최단 거리 짧은 정보 꺼내기
        val (currentVertex, currentDistance) = pq.poll()

        // 이미 처리된 노드
        if (distances[currentVertex] < currentDistance) continue

        // 현재 노드와 연결된 인접 노드 확인
        for (edge in graph[currentVertex]) {
            val nextDistance = currentDistance + edge.weight
            
            // 현재 노드를 거쳐 다른 도르오 이동하는 거리가 더 짧은 경우
            if (nextDistance < distances[edge.to]) {
                distances[edge.to] = nextDistance
                pq.offer(Pair(edge.to, nextDistance))
            }
        }
    }
    return distances
}
```

### 범위 및 함수 사용시 유의사항

- `indices` : `IntRange`타입을 반환하며, 수신객체의 전체 범위 반환. 위 예제 에선 0..6
- `until` : 범위의 마지막 요소를 포함하지 않음.
    - `0..5` => 0, 1, 2, 3, 4, 5
    - `0 until 5` => 0, 1, 2, 3, 4
- `numbers.size` , `numbers.lastIndex` 을 주의해서 사용.

### 시간 복잡도

- 간선의 수를 E 노드의 수를 V라고 했을 때 O`(E log V)`임.
- 우선순위 큐에서 꺼낸 노드는 연결된 노드만 탐색하므로 최악의 경우라도 총 간선 수인 E만큼 반복.
