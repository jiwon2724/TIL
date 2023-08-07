# RecyclerView

- 대량의 데이터를 효율적으로 표시할 수 있는 구성요소임.
- 이름에서 알 수 있듯 이는 요소들을 재활용함.
    - 스크롤되어 화면에 벗어나도 요소를 제거하지 않음.
    - 벗어난 요소를 재활용하여 새로운 뷰를 그려줌.

### 주요 클래스

- `RecyclerView` 는 `ViewGroup` 임. 즉, 다른 UI요소를 추가할 때 처럼 레이아웃에 추가하면 됨.
    - 데이터에 해당하는 `View` (요소)가 포함되므로 `ViewGroup` 임.
- 각 요소는 `ViewHolder` 객체로 정의됨. 이는 `RecyclerView` 위치에 대한 `ItemView`(요소)임
    - `ViewHolder` 는 `View`를 보관하는 객체임.
    - `ViewHolder`가 생성된 후, `RecyclerView`가 `ViewHolder`를 `View`에 바인딩함.
- 
- `RecyclerView`는 `Adapter`에서 메서드를 호출하여 데이터를 바인딩함.
    - `Adapter`는 `RecyclerView`에 표시되는 요소에 대한 바인딩을 제공함.
- `LayoutManager`는 `RecyclerView` 목록의 요소들을 정렬함.
    - 가로, 세로, 그리드를 지원함.

### Adapter, ViewHolder 구현

- 이 두 클래스가 함께 작동하여 데이터 표시 방식을 정의함.
- Adpater는 ViewHolder 객체를 만들며, 이러한 View에 데이터를 설정함.
    - 뷰에 데이터를 연결하는 프로세스가 바인딩임.
- Adapter 정의시 세 가지 메서드를 재정의 해야함.

```kotlin
class CustomAdapter(private val dataSet: Array<String>) : RecyclerView.Adapter<CustomAdapter.ViewHolder>() {
    override fun onCreateViewHolder(viewGroup: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(viewGroup.context).inflate(R.layout.text_row_item, viewGroup, false)
        return ViewHolder(view)
    }

    override fun onBindViewHolder(viewHolder: ViewHolder, position: Int) {
        viewHolder.textView.text = dataSet[position]
    }

    override fun getItemCount() = dataSet.size
}

class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
    val textView: TextView
}
```

- `onCreateViewHolder`
    - `ViewHolder`를 새로 만들 때 마다 이 메서드를 호출함.
    - `ViewHolder`에 연결된 View를 생성하되 바인딩은 하지 않음.
- `onBindViewHolder`
    - `ViewHolder` 를 데이터와 연결할 때 이 메서드를 호출함.
    - 데이터를 사용하여 `ViewHolder` 의 레이아웃을 채움.
- `getItemCount`
    - RecyclerView 데이터 목록의 크기를 가져올 때 이 메서드를 호출함.

### 호출 순서

1. `getItemCount` 함수가 호출되어 데이터 목록의 크기를 판단함.
2. `onCreateView` 에서 `ViewHolder` 를 생성함.
3. `onBindViewHolder` 를 호출하여 `ViewHolder` 에 데이터를 바인딩함.

### 참고목록
https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView
