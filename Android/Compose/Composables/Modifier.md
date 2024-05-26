# Modifier
- Surface 및 Text와 같은 대부분의 Compose UI 요소는 modifier 매개변수를 선택적으로 허용합니다.
- 상위 요소 레이아웃 내에서 컴포저블이 배치 및 표시되고 동작하는 방식을 컴포저블에 알려줍니다.
- 정렬, 애니메이션 처리, 배치, 클릭 가능 여부 또는 스크롤 가능 여부 지정, 변환 등에 사용할 수 있는 수십 가지의 Modifier가 있습니다.

```kotlin
@Composable
fun Greeting(name: String, modifier: Modifier = Modifier.padding(32.dp)) {
    Surface(color = MaterialTheme.colorScheme.primary) {
        Text(
            text = "Hello $name!",
            modifier = modifier // 상위 요소의 modifier -> padding 32dp
        )
    }
}
```

```kotlin
@Composable
fun Greeting(name: String, modifier: Modifier = Modifier.padding(32.dp)) {
    Surface(color = MaterialTheme.colorScheme.primary) {
        Text(
            text = "Hello $name!",
            modifier = modifier.padding(20.dp) // 상위 요소의 modifier -> padding 32dp + 하위 컴포저블이 지정한 패딩 20dp = 52dp
        )
    }
}
```
