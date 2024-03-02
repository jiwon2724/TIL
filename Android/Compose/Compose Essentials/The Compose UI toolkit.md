
# The Compose UI toolkit

> 본 포스팅은 아래 **Compose essentials** course를 학습하고 정리한 포스팅 입니다.
> 

[Jetpack Compose for Android Developers - Compose essentials](https://developer.android.com/courses/pathways/jetpack-compose-for-android-developers-1)

- 안드로이드 Compose는 UI toolkit과 함께 기본으로 제공되며, 풍부한 UI와 상호작용 됩니다.
- Compose를 사용하면 처음부터 Material Design을 구현하여 앱에 일관된 디자인과 느낌을 쉽게 제공할 수 있습니다.
    - Material Design은 UI 디자인의 모범 사례를 지원하는 구성요소 및 도구 시스템입니다.
    - Compose는 Material Design의 차세대 버전인 M2와 M3를 지원합니다.
        - M은 Material Design의 약자입니다.

## Material Design

- Material Design을 사용하면 colorScheme, typography, shapes을 모두 한곳에서 제공하여 브랜드에 맞게 앱 테마를 설정할 수 있습니다.

```kotlin
@Composable
fun JetsurveyTheme(
    useDarkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colors = if (!useDarkTheme) {
        LightColors
    } else {
        DarkColors
    }
    
    val systemUiController = rememberSystemUiController()
    DisposableEffect(systemUiController, useDarkTheme) { .. }
    
    MaterialTheme(
        colorScheme = colors,
        shapes = Shapes,
        typography = Typography,
        content = content
    )
}
```

- 위 코드는 Composable 기능을 사용하고, 색상, 모양, 글자에 대한 맞춤 값을 제공합니다.
- 시스템이 밝거나 어둡게 설정되어 있는지에 따라 다른 색상 값이 제공됩니다.
    - 이를 통해 사용자가 시스템 테마를 변경할 때 앱 색상이 자동으로 반응할 수 있습니다.
- `Typography` 속성에 따라서 글씨의 두께, 크기, 줄 높이 등 다양한 글꼴과 스타일을 설정할 수 있습니다.
    - 텍스트가 일관되게 표시되고 스타일이 지정될 수 있습니다.
- `Shapes` 속성으로 둥근 모서리(radius)를 정의할 수 있습니다.
- 커스텀 테마를 사용하려면 해당 테마 함수가 가장 바깥쪽 함수여야 합니다.
    - 해당 테마 안에 요소들은 정의한 속성(모양, 글자, 색상)을 사용하게 됩니다.

### Scaffold

- 기본 Material Design 구성요소를 `Scaffold` 라고 합니다.
- `Scaffold` 는 상단 앱바와 플로팅 작업 버튼이 있는 화면과 같은 일반적인 패턴으로 Material 구성요소를 배열하기 위한 기본 레이아웃 입니다.

## Suface

- `Surface` 는 Material Design의 또 다른 기본 구성요소입니다.
    - 이는 콘텐츠가 위치하는 Material Design의 중심 은유(metaphor)입니다.
- `Surface` 에는 색상, 모양, 테두리, 그림자, 색조, 높이 등을 속성으로 가지고 있습니다.

## Layouts

- Compose에는 세 가지 표준 레이아웃 요소가 있습니다.
    - Row
    - Column
    - Box
- 표준 레이웃은 매개변수도 허용하므로 배치 방법을 추가로 제어할 수 있습니다.

### Row

- 항목을 가로로 정렬하기 위해 UI 구성요소를 구성 가능한 함수로 래핑할 수 있습니다.

```kotlin
Row {
    Component1()
    Component1()
    Component1()
}
```

### Column

- 항목을 수직으로 정렬 할 수 있습니다.

```kotlin
Column {
    Component1()
    Component1()
    Component1()
}
```

### Box

- 요소를 다른 요소 위에 올리거나 쌓을 수 있습니다.

```kotlin
Box {
    Component1()
    Component1()
    Component1()
}
```

## Modifiers

- 크기 조정, 패딩 및 기타 모양과 같은 항목을 제어합니다.
    - 이는 Composable을 장식하거나 강화할 수 있습니다.
- 모든 구성 가능한 함수는 `Modifier` 를 매개 변수로 허용합니다.
- `Modifier` 를 사용하면 배경색 설정, 크기 변경, 패딩 추가 클릭 등 모든 종류의 작업을 수행할 수 있습니다.
- 구성 가능한 함수의 범위에 따라 추가 수정자를 사용할 수도 있습니다.

```kotlin
Box(Modifier.size(150.dp)) {
    Text(
        "Hello Compose!",
        Modifier.align(
            Alignment.BottomEnd
        ) 
    )
}
```

### 궁금한 부분

1. Scaffold는 Top, Mid, Bottom의 요소를 구성하기 위한 API 느낌?

```kotlin
Scaffold(
    topBar = { ... },
    content = { ... },
    bottomBar = { ... }
)
```

위 코드처럼 구성되어 있어서 레이아웃을 다르게 만들고 싶다면 사용 안될까요?

1. Composable은 보통 함수에 @Composable이 붙은 구성 가능한 함수를 뜻할까요? -> 맞음!
2. Modifier가 약간 XMl에서 color, width, height, padding등 짬뽕탕 느낌? -> 속성들은 Modifier의 확장함수로 되어있음.
3. 컴포즈에서 구성 가능한 함수는 파스칼 케이스? -> 명사의 의미를 가지고 있어서 컴포즈는 파스칼 케이스로! 네이밍도 명사로 지을 것.
4. 컴포즈로 UI를 구성하면 코드 라인이 길어질 수 밖에 없는 부분일까요? -> 넹.. 잘 쪼개서 개발하기
