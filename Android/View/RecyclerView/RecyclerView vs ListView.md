# RecyclerView vs ListView

- ListView도 뷰를 재활용해서 사용한다.
    - `getView` 함수에서 `convertView`를 활용
        - `convertView` 를 사용하지 않을경우, `getView` 함수가 호출될 때 마다 뷰를 새로 inflate함.
        - `convertView` 가 null이 아닌 경우는, 재사용 하는 경우이므로 데이터 매핑해주는 로직을 수행함.

```kotlin
override fun getView(position: Int, convertView: View?, parent: ViewGroup): View? {
        var view = convertView
        if (view == null) {
            Log.d("뷰 재활용 해주세요~~! : ", position.toString())
            view = LayoutInflater.from(parent.context).inflate(R.layout.list_item_person, parent, false)
        }
        val item = getItem(position)
        val nameView = view?.findViewById<TextView>(R.id.tv_name)
        nameView?.text = item.name

        val ageView = view?.findViewById<TextView>(R.id.tv_age)
        ageView?.text = item.age.toString()
        return view
    }
```

- view는 재활용 하지만, 데이터를 매핑해주는 단계에서 `findViewById` 를 계속 호출함.
    - view의 계층구조가 복잡할수록 비용이 많이 발생함.
        - `findViewById` 는 뷰의 계층 구조를 탐색하여 지정된 id를 가진 뷰를 찾아 반환함.
            - 즉, 계층 구조가 복잡할 수록 탐색하는 비용이 커짐.
- ViewBinidng, Databinding을 활용.
    - 바인딩 코드가 컴파일 시간 때 생성되므로, id를 탐색할 필요가 없음. 이는 `findViewById` 보다 빠르게 동작함.
- `ViewHolder` 패턴을 도입.
  - `ViewHolder` 패턴은 View의 객체를 ViewHolder에 보관하고, `findViewById`호출을 줄여 성능개선한 패턴임.

```kotlin
위와 같은 방법으로 ListView의 View 렌더링 방식은 성능 향상이 가능함.
```

### 그럼에도 왜 RecyclerView를 사용할까?

- `ListView` 에서 `ViewHolder` 패턴을 사용하는건 필수사항이 아닌 옵셔널임.
    - `ReyclerView`에선 `ViewHolder` 패턴을 강제함.
    - convertView와 ViewHolder 패턴을 사용 안한다면 성능적으로 보다 안좋음.
- `LayoutManager` 를 도입한 UI에 유연성을 부여함.
    - Horizontal, Grid 등 다양한 레이아웃을 손쉽게 구현이 가능.
- 기본적인 추가, 삭제, 이동에 대한 애니메이션을 제공함.
- 아이템 데코레이터나 아이템 터치 헬퍼와 같은 추가적인 유틸리티 클래스를 제공함.
    - 기능 확장성
