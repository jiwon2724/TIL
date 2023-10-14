# ****ConstraintLayout vs Other Layouts****

[https://medium.com/@wodbs135/번역-constraintlayout-performance-ba92200670e](https://medium.com/@wodbs135/%EB%B2%88%EC%97%AD-constraintlayout-performance-ba92200670e)

위 포스팅을 기반으로 여러가지 질문을 통하여 지식을 습득한 것을 정리한 게시글 입니다.

### 뷰 계층 평탄화

- 계층화가 심해지면 Measure, Layout 단계에서의 작업이 증가하여 성능이 떨어짐.
- `ConstraintLayout`은 간단한 제약조건으로 평탄한 계층구조를 만듦.

### 레이아웃 배치문제

- ViewGroup을 통하여 UI를 View계층에 중첩할 수 있음. 이러한 중첩은 비용을 부과할 수 있음.
- 루트 레이아웃부터 모든 하위 항목에 레이아웃 패스 프로세스를 진행하게됨.
    - Measure Pass, Layout Pass
    - 계층의 depth가 높아지고 복잡해지면 이로 인한 오버헤드가 증가할 것임.

### 이중 과세

- Android Framework는 measure, layout 단계를 매우 빠르게 실행함.
- 일부 복잡한 레이아웃에선 요소를 배치하기 전에 여러번 반복해야 해결할 수 있는 계층 구조도 있음.
    - RelativeLayout으로 예시를 들어보자면
        1. 루트 레이아웃 부터 하위 개체 까지 measure, layout 단계를 실행함.
        2. 해당 정보를 사용해 가중치를 고려하여 연관된 뷰의 적절한 위치를 파악함.
            1. RelativeLayout은 다른 View 위치를 기준으로 View를 배치할 수 있는 레이아웃임.
        3. 레이아웃 단계를 수행하여 개체의 위치를 완성함.
        4. 렌더링 후 배치해야하는 곳으로 이동.
    - 즉, 위 과정은 RelativeLayout에서 뷰A가 뷰B 아래에 위치해야 한다고 가정한다면, 뷰B 위치와 높이를 먼저 알아야 뷰A의 정확한 위치를 결정할 수 있음.
- 다중 Measure-Layout(이중 과세) 단계까 그 자체로 성능 저하를 가져오진 않음.
    - 올바르지 못한 곳에 배치된다면 그럴 가능성이 있는 것임.
    - 다음 조건중 하나가 컨테이너(ViewGroup)에 적용되는 경우는 주의할 것.
        - 뷰 계층 상위 요소일 때(ViewGroup을 중첩시킬 때 인 듯)
        - 뷰 계층이 깊을 때
        - RecyclerView 처럼 뷰를 구성할 때
- 즉, 뷰 계층을 평탄화 하고, 레이아웃 중첩을 지양 하는 것이 좋음.

### ConstraintLayout vs Other Layouts

ConstraintLayout과 다른 레이아웃간 View 배치에 대해서 성능비교 결과를 알아보자.

1. **다른 뷰의 중앙에 새로운 뷰를 배치하는 경우**
    1. FrameLayout > ConstraintLayout > LinearLayout
2. **100개의 아이템을 가진 RecyclerView**
    1. ConstraintLayout > RelativeLayout > LinearLayout
3. **TextView와 Image로만 이루어진 간단한 UI**
    1. LinearLayout > ConstraintLayout
        1. 아주 간단한 UI에선 LinearLayout 성능이 더 좋음.
        2. 1 depth에 자식 뷰의 위치, 크기를 구해야 하는 경우가 아닌 배치만 하는 경우라면 굳이 ConstraintLayout를 사용할 필요는 없음.
4. **Image, Text, ScrolView, Button 등 으로 이루어진 복잡한 UI**
    1. ConstraintLayout > Other Layouts

### 참고 사항

- `ConstraintLayout` 가 절대적으로 성능이 우세한 것은 아님.
    - 참고한 블로그의 마지막 섹션 확인
- 렌더링 성능에 영향을 미치는 요소 중 이중 과세로 등의 이유로 평탄한 뷰를 가진 `ConstraintLayout` 이 이점을 갖음.
    - 간단한 제약조건으로 평탄한 구조를 만듦.
    - View의 수가 많아질 수록 렌더링 성능엔 이점임.
    - 이는 100개의 아이템을 가진 RecyclerView 성능과 직결됨.
- ViewGroup 내부에 ViewGroup을 사용하는 것을 지양해야함.
    - 어쩔수 없이 사용해야 한다면 `ConstraintLayout` 내부에 편의성에 따라 다른 레이아웃을 취사선택 하는 것이 좋음.
        - `group`, `flow`를 응용해서 묶는것도 좋음.
            - 이는 가상 레이아웃이므로 레이아웃 계층에 영향을 주지 않음.
        - `ConstraintLayout` 내부에 `ConstraintLayout` 사용은 지양해야함.