
# **Accelerate with tooling**

- Compose를 사용하면 아름다운 UI를 빠르게 구축할 수 있습니다.
- Android Studio Tool과 함께 개발 프로세스를 더욱 가속화할 수 있습니다.

## Accelerate Development

- Composable 작성을 시작하려면 `comp`를 작성하고 Tab또는 Return키를 누르면 자동으로 Composable이 생성됩니다.
![화면-기록-2024-03-05-오후-11 41 58](https://github.com/jiwon2724/TIL/assets/70135188/3fe690c1-7586-44e9-9de8-869fe7256258)


- 위와 같게 Row를 구성하려면 `WR`을 입력하고 Tab키나 Return키를 눌러 Row를 실행하면 됩니다.
- 위 같은 코드 조각을 생성하는데 도움이 되는 것을 `Live Templete`이라고 합니다.
    - 이는 Preference(Settings) → Editor → Live Templates에 있습니다.
- 이미지를 삽입할 때(Drawable 작업) 미리보기 아이콘으로 빠르게 미리보고 다른 이미지로 전환할 수 있습니다.
![스크린샷 2024-03-05 오후 11 51 44](https://github.com/jiwon2724/TIL/assets/70135188/91e5634b-8bf1-437b-87ed-0b836b38726d)



- 위 아이콘 미리보기에서 아이콘을 클릭하면 다음과 같은 drawable 파일들이 렌더링 됩니다.
![스크린샷 2024-03-05 오후 11 53 42](https://github.com/jiwon2724/TIL/assets/70135188/644534fc-7712-461d-ba65-f490c84ef2f3)


- 마찬가지로 Color를 지정하는 API에서도 위처럼 적용된 색상을 클릭하여 색상을 빠르게 변경할 수 있습니다.
![스크린샷 2024-03-05 오후 11 55 44](https://github.com/jiwon2724/TIL/assets/70135188/4dbabbcd-f71c-4f40-81c5-22a1783c0902)

- 라이브 템플릿과 아이콘은 실수를 줄이면서 코드를 작성하는데 도움이 됩니다.

### Preview
- 미리보기를 사용하여 UI를 빠르게 확인하는 방법을 살펴보겠습니다.
- 위와 같이 p-r-e-v를 입력하고 Tab, Return키를 누르면 @Preview Composable이 만들어집니다.
  - 미리보기 컴포저블은 파라미터를 허용할 수 없습니다.
 
![스크린샷 2024-03-06 오후 9 43 24](https://github.com/jiwon2724/TIL/assets/70135188/a618defe-6368-49c6-851b-e4138016783c)

- @Preview 어노테이션 옆 톱니바퀴 버튼으로 해당 미리보기의 모든 종류의 속성을 지정할 수 있습니다.
  - ex) 다크모드를 사용하여 특정 배경색을 미리볼 수 있습니다.
- 동일한 미리보기에 @Preview 어노테이션을 여러번 추가할 수 있으므로 여러 타입의 UI를 미리 볼 수 있습니다.

### Multi Preview
- 다중 미리보기 기능을 사용하면 UI에 다양한 속성의 미리보기를 확인할 수 있습니다.
- annotation class를 정의하여 해당 확인하고 싶은 미리보기를 정의해주면 됩니다.
- 다중 미리보기를 사용하면 다양한 미리보기가 자동으로 표시됩니다.
```kotlin
@Preview(
    name = "Small font",
    group = "Font scales",
    fontScale = 0.5f
)
@Preview(
    name = "Large font",
    group = "Font scales",
    fontScale = 1.5f
)
annotation class FontScalePreviews

@FontScalePreviews // annotation class사용
@Composable
fun SurveyAnswerPreview() {
    ..
}
```

### 실시간 편집 기능
- 에뮬레이터나 디바이스에서 어떻게 작동하는지 확인하려면 여백에 있는 실행 아이콘을 사용하면 됩니다.
- 안드로이드 스튜디오에서 입력한 변경사항이 즉시 업데이트 됩니다.
