
# @AndroidEntryPoint의 이해

## Component vs SubComponent

- Dagger2에서 말하는 `Component`는 의존성을 관리하는 컨테이너를 의미합니다.
    - `SubComponent`는 `Component`의 하위에 속하는 `Component` 를 의미합니다.
    - `SubComponent` 의 속하는 또다른 하위 컴포넌트도 `SubComponent` 입니다.
- Hilt에서 `Component` 는 `SingletonComponent` 하나만 존재합니다.
    - 나머지 컴포넌트들은 전부 `SubComponent` 입니다.
        - ex) ActivityRetainedComponenet, ActivityComonenet, FragmentComponent 등
        - 즉, `SingletonComponent` 를 제외한 컴포넌트들은 `SingletonComponent` 의 하위 `Component` 입니다.
- Hilt에서 서브 컴포넌트를 인스턴스하는 방법은 안드로이드 클래스에 `@AndroidEntryPoint`를 선언하면 됩니다.

## @AndroidEntryPoint가 지원하는 타입

- `@HiltAndroidApp` 어노테이션을 통해 의존성 주입을 가능하게 만들었다면, 안드로이드 클래스에서도 의존성 주입을 활성화 시킬 수 있습니다.
    - 이때 `@AndroidEntryPoint`를 사용합니다.
- @AndroidEntryPoint는 다음과 같은 클래스 타입을 지원합니다.
    - Activity
    - Fragment
    - View
    - Service
    - BroadcastReceiver
        - 위 클래스 타입과는 다르게 `BroadcastReceiver` 는 표준 컴포넌트를 가지고 있지 않으므로 `SingletonComponent`를 통해서 의존성을 주입 받으면 됩니다.

## @AndroidEntryPoint 사용시 주의사항

1. Fragment와 같이 Activity가 반드시 존재해야 하는 상황에서 상위, 하위 컴포넌트 둘 다 어노테이션을 설정해야합니다.
    1. 즉, 안드로이드 클래스에서 의존성 주입 시, 상위(서브)컴포넌트에도 반드시 @AndroidEntryPoint를 선언해야 합니다.

```kotlin
@AndroidEntryPoint
class MyActivity : ComponentActivity() {
    // MyFragment 실행
}

@AndroidEntryPoint
class MyFragment : Fragment() {
    // 의존성 주입
}
```

1. Hilt는 ComponentActivity를 상속한 Activity만 지원합니다.
    1. 즉, ComponentActivity를 상속하고 있는지 확인해야합니다.
2. Hilt는 androidx의 Fragment만 지원합니다.
    1. Android SDK에 포함된 Fragment는 Deprecated 됐으며, Hilt를 지원하지 않습니다.

## Retained Fragment

- `retainInstance`는 Fragment의 인스턴스를 유지를 결정하는 메서드입니다.
    - Depecated 되었습니다.
- `retainInstance` 메서드가 true인 경우, 구성 변경에서도 Fragment가 유지됩니다.
- 하지만 Fragment가 Hilt와 함께 사용되는 경우라면, 컴포넌트에 바인딩된 인스턴스가 유지가 되지 않습니다.
    - 생명주기에 따른 힐트 컴포넌트의 주입되는 프로세스가 설정이 되지 않습니다.
        - `retainInstance` 는 Fragment 생명주기의 `onDestory()`는 호출되지 않고, `onDetach()` 는 호출됩니다.
        - 즉,  Hilt가 기대하는 생명주기 이벤트가 발생하지 않게 됩니다. 이는 Hilt가 정상적으로 의존성을 주입하고 관리하는 프로세스에 영향을 줄 수 있습니다.
    - 의존성들을 `retainInstance` 을 통해 유지하려고 한다면 메모리 누수가 발생할 수 있습니다.
- 상위 컴포넌트에 데이터를 저장하거나, SingletonComponent 혹은 ViewModel로 Fragment의 인스턴스를 유지할 수 있습니다.
    - 이는 메모리를 관리하는 측면이나, 코드를 유지보수 하는 경우에도 좋은 방법입니다.
 
```kotlin
@AndroidEntryPoint
class PurchaseFragment : Fragment() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        retainInstance = true // 이렇게 사용하지 마세요!
    }
}
```
