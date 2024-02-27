
# **Implement a real-world design**

> 본 포스팅은 아래 **Compose essentials** course의 codelab을 학습하고 정리한 포스팅 입니다.
> 

[Compose의 기본 레이아웃  |  Android Developers](https://developer.android.com/codelabs/jetpack-compose-layouts?continue=https://developer.android.com/courses/pathways/jetpack-compose-for-android-developers-1&hl=ko#0)

- UI toolkit Compose를 사용하면 앱 디자인을 쉽게 구현할 수 있습니다.
- 개발자가 UI의 디자인을 기술하면 Compose가 화면에 그리는 작업을 처리합니다.
- 디자인을 구현하라는 요청을 받았을 땐, 먼저 디자인의 구조를 명확하게 파악하는게 중요합니다.
    - 곧바로 코딩을 시작하는 대신 디자인을 먼저 분석해 보세요.
    - 해당 UI를 여러 개의 재사용 가능한 부분으로 나누려면 어떻게 해야 할까요?
- 디자인 분석이 완료됐다면, 컴포저블 구현을 시작할 수 있습니다. 가장 낮은 컴포저블부터 시작한 다음 이를 조합하여 복잡한 컴포저블을 구현하는게 좋습니다.

## SearchBar

- 입력창을 구현하려면 `TextField` 라는 Material 구성요소를 사용합니다.
- Compose Material 라이브러리에는 이 Material 구성요소의 구현인 `TextField` 컴포저블이 있습니다.

```kotlin
@Composable
fun SearchBar(
    modifier: Modifier = Modifier
) {
    TextField(
        value = "",
        onValueChange = { },
        leadingIcon = { //  다른 컴포저블을 받는 매개변수
            Icon(
                imageVector = Icons.Default.Search,
                contentDescription = null
            )
        },
        colors = TextFieldDefaults.colors( // 특정 컬러 재정의
            unfocusedContainerColor = MaterialTheme.colorScheme.surface,
            focusedContainerColor = MaterialTheme.colorScheme.surface
        ),
        placeHolder = {
            Text(stringResource(R.string.placeholder_search))
        },
        modifier = modifier
            .fillMaxWidth() // 상위 요소의 전체 가로 공간을 차지함
            .heightIn(min = 56.dp) // 특정 최소 높이 지정 (글꼴 확대 시 커질 수 있음)
    )
}
```

- `modifier` 의 매개변수를 받아 `TextField` 에 전달하는 방식은 Compose 가이드 라인에 따른 권장사합 입니다.
- 이 방식을 사용하면 메서드의 호출자가 컴포저블의 디자인과 분위기를 수정할 수 있어 유연성이 높아지고, 재사용이 가능하게 됩니다.

## **AlignYourBodyElement**

- `Column` 을 통하여 컴포저블을 세로 방향으로 배치할 수 있습니다.

```kotlin
@Composable
fun AlignYourBodyElement(
    @DrawableRes drawable: Int,
    @StringRes text: Int,
    modifier: Modifier = Modifier
) {
    Column(
        horizontalAlignment = Alignment.CenterHorizontally, // 정렬
        modifier = modifier
    ) {
        Image( // Image 컴포저블 조정
            painter = painterResource(id = R.drawable.ab1_inversions),
            contentDescription = null,
            contentScale = ContentScale.Crop,
            modifier = Modifier
                .size(88.dp)
                .clip(CircleShape)
        )
        Text(
            text = stringResource(text),
            modifier = Modifier.paddingFromBaseline(top = 24.dp, bottom = 8.dp),
            style = MaterialTheme.typography.bodyMedium
        )
    }
}

@Preview(showBackground = true, backgroundColor = 0xFFF5F0EE)
@Composable
fun AlignYourBodyElementPreview() {
    MySootheTheme {
        AlignYourBodyElement(
            text = R.string.ab1_inversions,
            drawable = R.drawable.ab1_inversions,
            modifier = Modifier.padding(8.dp)
        )
    }
}
```

## FavoriteCollectionCard

- 전체 화면의 배경과 다른 surfaceVariant를 배경 색상을 사용합니다.
- Material의 `Surface` 컴포저블을 사용합니다.

```kotlin
@Composable
fun FavoriteCollectionCard(
   @DrawableRes drawable: Int,
   @StringRes text: Int,
   modifier: Modifier = Modifier
) {
   Surface(
       shape = MaterialTheme.shapes.medium,
       color = MaterialTheme.colorScheme.surfaceVariant,
       modifier = modifier
   ) {
       Row(
           verticalAlignment = Alignment.CenterVertically,
           modifier = Modifier.width(255.dp)
       ) {
           Image(
               painter = painterResource(drawable),
               contentDescription = null,
               contentScale = ContentScale.Crop,
               modifier = Modifier.size(80.dp)
           )
           Text(
               text = stringResource(text),
               style = MaterialTheme.typography.titleMedium,
               modifier = Modifier.padding(horizontal = 16.dp)
           )
       }
   }
}
```

## AlignYourBodyRow

- `LazyLow`, `LazyColumn` 컴포저블을 사용하여 스크롤 가능한 행,열 을 구현할 수 있습니다.
- `Lazy` 컴포저블은 모든 요소를 동시에 렌더링하는 대신 화면에 표시되는 요소만 렌더링하여 앱의 성능을 유지합니다.
- `Lazy` 컴포저블의 하위 요소는 컴포저블이 아닙니다.

```kotlin
import androidx.compose.foundation.layout.PaddingValues

@Composable
fun AlignYourBodyRow(
   modifier: Modifier = Modifier
) {
   LazyRow(
       horizontalArrangement = Arrangement.spacedBy(8.dp), // 컴포저블 사이 고정 공간 추가
       contentPadding = PaddingValues(horizontal = 16.dp),
       modifier = modifier
   ) {
       items(alignYourBodyData) { item -> // List
           AlignYourBodyElement(item.drawable, item.text)
       }
   }
}
```

## 궁금사항

1. `fillMaxWidth` 는 “상위 요소의 전체 가라 공간을 차지한다” 여기서 상위요소는 SearchBar를 호출한 컴포저블?
2. 컴포저블 import가 잘 안되는 것 같은데 팁이 있을까요?.?

3.

```kotlin
@Preview(showBackground = true, backgroundColor = 0xFFF5F0EE)
@Composable
fun AlignYourBodyElementPreview() {
    MySootheTheme {
        AlignYourBodyElement(
            text = R.string.ab1_inversions,
            drawable = R.drawable.ab1_inversions,
            modifier = Modifier.padding(8.dp)
        )
    }
}
```

상위 컴포넌트에서 패딩을 설정해주면, 자식 컴포넌트에서도 패딩값이 적용?

1. Surface는 정확히 언제 사용해야할까요..? 머터리얼 디자인을 사용하는 백그라운드?
