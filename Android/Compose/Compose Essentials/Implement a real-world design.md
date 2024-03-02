
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

## SearchBar - 수정자(Modifier)

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

## AlignYourBodyElement - 정렬

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

## FavoriteCollectionCard - Surface

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

## AlignYourBodyRow - 배치

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
## FavoriteCollectionsGrid - Lazy Grid
- `LazyRow`와 `Column`을 이용하여 Grid한 구조를 만들 수 있습니다.
- `LazyHorizontalGrid`는 항목 - Grid 요소를 더 효과적으로 지원합니다.

```kotlin
@Composable
fun FavoriteCollectionsGrid(
   modifier: Modifier = Modifier
) {
   LazyHorizontalGrid(
       rows = GridCells.Fixed(2),
       contentPadding = PaddingValues(horizontal = 16.dp),
       horizontalArrangement = Arrangement.spacedBy(16.dp),
       verticalArrangement = Arrangement.spacedBy(16.dp),
       modifier = modifier.height(168.dp)
   ) {
       items(favoriteCollectionsData) { item ->
           FavoriteCollectionCard(item.drawable, item.text, Modifier.height(80.dp))
       }
   }
}
```

## HomeSection - 슬롯 API
- 동일한 패턴을 따르는 여러 개의 섹션은 슬롯 API를 사용하면 유연하게 처리가 가능합니다.

```kotlin
@Composable
fun HomeSection(
   @StringRes title: Int,
   modifier: Modifier = Modifier,
   content: @Composable () -> Unit
) {
   Column(modifier) {
       Text(
           text = stringResource(title),
           style = MaterialTheme.typography.titleMedium,
           modifier = Modifier
               .paddingFromBaseline(top = 40.dp, bottom = 16.dp)
               .padding(horizontal = 16.dp)
       )
       content()
   }
}
```

## HomeScreen - 스크롤
- `Spacer`를 사용하면 `Column`내부에서 더 많은 공간을 확보할 수 있습니다.
- `Lazy`레이아웃은 자동으로 스크롤 동작을 추가합니다. 하지만 항상 `Lazy`레이아웃이 필요한 것은 아닙니다.
  - 리스트에 포함된 요소가 많거나 로드해야하는 데이터 set이 많은 경우에 사용합니다.
- 요소의 개수가 많지 않은 경우에는 간단한 `Column`또는 `Row`를 사용하고, 스크롤 동작을 수동으로 추가하면 됩니다.
  - `verticalScroll`
  - `horizontalScroll`

```kotlin
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.verticalScroll

@Composable
fun HomeScreen(modifier: Modifier = Modifier) {
   Column(
       modifier
           .verticalScroll(rememberScrollState())
   ) {
       Spacer(Modifier.height(16.dp))
       SearchBar(Modifier.padding(horizontal = 16.dp))
       HomeSection(title = R.string.align_your_body) {
           AlignYourBodyRow()
       }
       HomeSection(title = R.string.favorite_collections) {
           FavoriteCollectionsGrid()
       }
       Spacer(Modifier.height(16.dp))
   }
}
```

## SootheBottomNavigation - 하단 탐색
- `NavigationBar`컴포저블을 사용하여 여러 화면 간 전환할 수 있는 탐색메뉴를 사용할 수 있습니다.
- 하나 이상의 `NavigationBarItem`을 추가 해야합니다.

```kotlin
@Composable
private fun SootheBottomNavigation(modifier: Modifier = Modifier) {
   NavigationBar(
       containerColor = MaterialTheme.colorScheme.surfaceVariant,
       modifier = modifier
   ) {
       NavigationBarItem(
           icon = {
               Icon(
                   imageVector = Icons.Default.Spa,
                   contentDescription = null
               )
           },
           label = {
               Text(stringResource(R.string.bottom_navigation_home))
           },
           selected = true,
           onClick = {}
       )
       NavigationBarItem(
           icon = {
               Icon(
                   imageVector = Icons.Default.AccountCircle,
                   contentDescription = null
               )
           },
           label = {
               Text(stringResource(R.string.bottom_navigation_profile))
           },
           selected = false,
           onClick = {}
       )
   }
}
```

## MySootheAppPortrait - Scaffold
- `Scaffold`는 머터리얼 디자인을 구현하는 앱을 위한 구성 가능한 최상위 수준 컴포저블을 제공합니다.
- 이는 다양한 머터리얼 개념의 슬롯이 포함되어 있습니다.
```kotlin
import androidx.compose.material3.Scaffold

@Composable
fun MySootheAppPortrait() {
   MySootheTheme {
       Scaffold(
           bottomBar = { SootheBottomNavigation() }
       ) { padding ->
           HomeScreen(Modifier.padding(padding))
       }
   }
}
```

## SootheNavigationRail - 탐색레일
- 하단 탐색이 화면 콘텐츠 왼쪽 레일로 전환되는 방식은 `NavigationRail`을 사용하면 됩니다.
- `NavigationBar`와 마찬가지로 `NavigationRailItem`요소를 추가하면 됩니다.

```kotlin
import androidx.compose.foundation.layout.fillMaxHeight

@Composable
private fun SootheNavigationRail(modifier: Modifier = Modifier) {
   NavigationRail(
       modifier = modifier.padding(start = 8.dp, end = 8.dp),
       containerColor = MaterialTheme.colorScheme.background,
   ) {
       Column(
           modifier = modifier.fillMaxHeight(),
           verticalArrangement = Arrangement.Center,
           horizontalAlignment = Alignment.CenterHorizontally
       ) {
           NavigationRailItem(
               icon = {
                   Icon(
                       imageVector = Icons.Default.Spa,
                       contentDescription = null
                   )
               },
               label = {
                   Text(stringResource(R.string.bottom_navigation_home))
               },
               selected = true,
               onClick = {}
           )
           Spacer(modifier = Modifier.height(8.dp))
           NavigationRailItem(
               icon = {
                   Icon(
                       imageVector = Icons.Default.AccountCircle,
                       contentDescription = null
                   )
               },
               label = {
                   Text(stringResource(R.string.bottom_navigation_profile))
               },
               selected = false,
               onClick = {}
           )
       }
   }
}
```
- 세로 버전은 `Scaffold`를 사용했지만, 가로 모드는 Row와 탐색 레일, 화면 콘텐츠를 나란히 배치하면 됩니다.

```kotlin
@Composable
fun MySootheAppLandscape() {
   MySootheTheme {
       Row {
           SootheNavigationRail()
           HomeScreen()
       }
   }
}
```
## MySootheApp - 화면 크기
- 기기나 에뮬레이터에서 옆으로 돌리면 가로모드 버전이 표시되지 않습니다.
- 이는 언제 어떤 앱의 구성을 표시할지 앱에 알려줘야 하기 때문입니다.
- `calculateWindowSizeClass()` 함수를 사용하여 디바이스의 구성을 확인합니다.

```kotlin
import androidx.compose.material3.windowsizeclass.WindowSizeClass
import androidx.compose.material3.windowsizeclass.WindowWidthSizeClass
@Composable
fun MySootheApp(windowSize: WindowSizeClass) {
   when (windowSize.widthSizeClass) {
       WindowWidthSizeClass.Compact -> {
           MySootheAppPortrait()
       }
       WindowWidthSizeClass.Expanded -> {
           MySootheAppLandscape()
       }
   }
}
```
- `calculateWindowSize()`는 아직 실험 단계이므로 `ExperimentalMaterial3WindowSizeClassApi` 클래스를 선택해야 합니다.

```kotlin
import androidx.compose.material3.windowsizeclass.ExperimentalMaterial3WindowSizeClassApi
import androidx.compose.material3.windowsizeclass.calculateWindowSizeClass

class MainActivity : ComponentActivity() {
   @OptIn(ExperimentalMaterial3WindowSizeClassApi::class)
   override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       setContent {
           val windowSizeClass = calculateWindowSizeClass(this)
           MySootheApp(windowSizeClass)
       }
   }
}
```

## 궁금사항

1. `fillMaxWidth` 는 “상위 요소의 전체 가라 공간을 차지한다” 여기서 상위요소는 SearchBar를 호출한 컴포저블? -> 부모 레이아웃의 가로 공간을 차지함. 부모 레이아웃이 없다면, 디바이스의 최대 가로 공간임. match_parent와 비슷합니다!
2. 컴포저블 import가 잘 안되는 것 같은데 팁이 있을까요?.? -> X
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
상위 컴포넌트에서 패딩을 설정해주면, 자식 컴포넌트에서도 패딩값이 적용? -> 인자로 넘겨준 modifier를 사용해야 적용됨.

4. Surface는 정확히 언제 사용해야할까요..? 머터리얼 디자인을 사용하는 백그라운드? -> 자식 컴포저블의 디자인적 요소를 재활용 하는 느낌. ex) radius5dp의 백그라운드인 경우
5. Scaffold는 음.. 슬롯 API의 모음? 느낌일까요? -> 맞음!
