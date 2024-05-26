# Surface
- Surface 컴포저블은 컴포저블 요소를 감싸고 Material Design 시스템의 시각적 매개변수를 제공하는 데 사용됩니다
- Material Design 가이드라인을 따르는 UI 요소를 만들기 쉬워지며, 이를 통해 일관된 디자인을 유지할 수 있습니다.
- 배경색, 모양, 그림자, 엘리베이션 등의 속성을 쉽게 적용할 수 있게 해줍니다.
  - color, shape, elevation 등
 
```kotlin
@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    Surface(
        color = Color.Yellow,
        shape = RoundedCornerShape(16.dp),
        shadowElevation = 8.dp,
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
            .border(width = 2.dp, color = Color.Black, shape = RoundedCornerShape(8.dp))
    ) {
        Text(
            text = "Hello $name!",
            modifier = modifier,
        )
    }
}
```
