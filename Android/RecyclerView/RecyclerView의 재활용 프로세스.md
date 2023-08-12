### 동작 원리

1. 데이터 셋(데이터 리스트)에 데이터(아이템)이 보관됨.

```kotlin
val itemList: ArrayList<Item>

itemList : 데이터 셋(데이터 리스트)
Item : 데이터(아이템)
```

1. Adapter가 데이터를 뷰에 바인딩함.
2. 뷰를 제어하는 레이아웃 매니저에게 이를 제공함.

```kotlin
RecyclerView는 모든 아이템에 대해 아이템 뷰를 할당하지 않음. 화면에 맞는 아이템 뷰의 수만 할당하고
스크롤 할 때 마다 해당 아이템 레이웃을 재활용함. 뷰가 스크롤되어 보이지 않게 되면, **재활용 프로세스**를 거침.
```

### 재활용 프로세스

![스크린샷 2023-08-12 오후 9 44 13](https://github.com/jiwon2724/TIL/assets/70135188/477c8299-8e10-48c7-a864-b6b37ea5a44a)


1. 스크롤 시 뷰가 보이지 않고 더이상 표시되지 않으면 `Scrap View`가 됨.

```kotlin
Scrap View란 부모 RecyclerView에 붙어있지만, 제거되거나 재사용될 수 있는 뷰를 의미함.
```

1. RecyclerView가 스크롤되면서 화면 밖으로 벗어난 뷰는 `Scrap Heap`에 잠시 보관됨.
    1. `Scrap Heap` : 뷰가 일시적으로 저장되는 공간 
        1. 이는(`Scrap View`) 동일한 레이아웃을 그릴 때 재사용됨.
        2. 즉, 동일한 레이아웃이 재사용 될 때 어댑터를 통하지 않고 레이아웃 매니저로 직접 반환하여 렌더링함.
        3. 화면에 다시 들어올 가능성이 높은 뷰들을 빠르게 다시 화면에 표시하기 위해 임시 보관함.
2. 새로운 아이템을 표시할 경우 재사용하기 위해 `Recycled Pool` 에서 뷰를 가져옴.
    1. `Scrap Heap` 에 오랜시간 머무른 `Scrap View` 들은 `Recycled Pool` 로 옮겨짐.
    2. `Recycled Pool`에 있는 재활용 대기중인 뷰들을 `dirty view` 라고 부름.
        1. 뷰를 표시하기 전에 어댑터에 의해 다시 바인딩 되어야함.

```kotlin
Recycled Pool이란 오랜시간 사용하지 않은 뷰를 저장하고, 필요할 때 재사용하기 위한 Pool
```

1. `dirty view` 를 리바인딩하고, 화면에 재활용되어 렌더링됨.

### RecyclerView의 C**aching 메커니즘**

![image](https://github.com/jiwon2724/TIL/assets/70135188/1f2b8c8e-8f6a-444d-9198-9b6df8a3006e)

1. `LayoutManager` 가 `RecyclerView`에 포지션으로 `View` 를 요청함.
2. `RecyclerView` 는 포지션으로 캐시를 확인하여 있다면 이를 `LayoutManager` 에 반환.
3. 캐시에 존재하지 않다면 어댑터에게 viewType을 물어보고 해당 `Recycled Pool` 에 요청함.
    1. ViewType 마다 Pool이 할당됨.
4. `Recycled Pool` 에 존재한다면 이를 반환하고, 없다면 어댑터에 뷰홀더를 생성할 것을 요청함.
5. 뷰를 찾으면 어댑터에 뷰를 바인딩 해주고, `LayoutManager` 에게 이를 리턴함.

### 참고자료

- https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView.RecycledViewPool
- [https://cs.android.com/androidx/platform/frameworks/support/+/androidx-main:recyclerview/recyclerview/src/main/java/androidx/recyclerview/widget/RecyclerView.java?q=file:androidx%2Frecyclerview%2Fwidget%2FRecyclerView.java class:androidx.recyclerview.widget.RecyclerView.RecycledViewPool](https://cs.android.com/androidx/platform/frameworks/support/+/androidx-main:recyclerview/recyclerview/src/main/java/androidx/recyclerview/widget/RecyclerView.java?q=file:androidx%2Frecyclerview%2Fwidget%2FRecyclerView.java%20class:androidx.recyclerview.widget.RecyclerView.RecycledViewPool)
- https://www.slideshare.net/BoostCamp1/7-tech-talkrecyclerview
- [https://medium.com/@wodbs135/recyclerview를-더-잘-사용하는-방법-879cd52bcdba](https://medium.com/@wodbs135/recyclerview%EB%A5%BC-%EB%8D%94-%EC%9E%98-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95-879cd52bcdba)
- [https://medium.com/hongbeomi-dev/번역-recyclerview의-내부-동작-941a2827fa5a](https://medium.com/hongbeomi-dev/%EB%B2%88%EC%97%AD-recyclerview%EC%9D%98-%EB%82%B4%EB%B6%80-%EB%8F%99%EC%9E%91-941a2827fa5a)
