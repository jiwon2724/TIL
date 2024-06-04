# LazyLayout
- Column 컴포저블에 수천, 또는 그 이상의 수를 처리하기는 어렵습니다.
- 스크롤이 가능한 행, 열을 표시하기 위해 `LazyColumn`, `LazyLow`를 사용합니다.
  - 이는 Android View의 RecyclerView와 동일합니다.
- Lazy API는 범위 내에서 `items` 요소를 제공합니다.

```kotlin
@Composable
private fun Greetings(
    modifier: Modifier = Modifier,
    names: List<String> = List(1000) { "$it" }
) {
    LazyColumn(modifier = modifier.padding(vertical = 4.dp)) {
        items(items = names) { name ->
            Greeting(name = name)
        }
    }
}
```
> LazyXXX는 RecyclerView와 같은 하위 요소를 재활용하지 않습니다. 컴포저블을 렌더링 하는 것은 Android View를 인스턴스화 하는 것보다 상대적으로 비용이 적게들고, 스크롤 시 컴포저블을 렌더링하며 성능을 유지합니다.
