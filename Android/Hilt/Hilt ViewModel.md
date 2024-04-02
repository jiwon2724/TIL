## ViewModel

- 비즈니스 로직 또는 UI 상태를 갖는 홀더입니다.
- Activity, Fragment보다 오래 메모리에 살아남아 상태를 보존할 수 있는 저장소 역할도 합니다.

## 기본적인 ViewModel 인스턴스 획득

```kotlin
class FooViewModel: ViewModel()

val viewModel = ViewModelProvider(viewModelStoreOwner)[FooViewModel::class.java]
```

- Activity나 Fragment보다 오래살아 남기 위해선 `ViewModelProvider`를 통해서 인스턴스화를 진행해야합니다.

```kotlin
val viewModel: FooViewModel by viewModels()
```

- 코틀린에선 위임 패턴을 사용하여 ViewModel을 위와 같이 인스턴스화 할 수 있습니다.
- 내부적으론 `ViewModelProvider`를 활용하고, ViewModel을 인스턴스화 하는 과정이 위임 함수에 캡슐화 되어 있습니다.

## 매개변수가 있는 ViewModel 생성

```kotlin
class FooViewModel(foo: Foo, handle: SavedStateHandle): ViewModel()

val viewModel: FooViewModel by viewModels(
    factoryProducer = { 
        // ViewModelProvider.Factory 반환
    }
)
```

- ViewModel에게 인자를 전달하기 위해 `ViewModelProvider.Factory`를 미리 정의해야 합니다.

```kotlin
class FooViewModelFactory(val foo: Foo) : AbstractSavedStateViewModelFactory() {
    override fun <T: ViewModel> create(
        key: String,
        modelClass: Class<T>,
        handle: SavedStateHandle
    ): T {
        return FooViewModel(foo, handle) as T
    }
}
```

- ViewModel을 생성하는 팩토리를 정의하고, `create` 함수가 호출되면서 `foo`와 `handle`을 인자로 전달하여 반환합니다.

## ViewModel 인스턴스 획득 프로세스

- `create` 함수가 호출되는 과정을 도식화하면 다음과 같습니다.

![스크린샷 2024-04-02 오후 11 11 03](https://github.com/jiwon2724/TIL/assets/70135188/a115137d-164f-4f88-bcbe-f374f1a99b53)


- Activity나 Fragment는 `ViewModelStore`를 따로 관리하고 있어 Activity 생명 주기보다 긴 생명주기를 갖는 이유입니다.

## ViewModel 생명주기

![스크린샷 2024-04-02 오후 11 13 35](https://github.com/jiwon2724/TIL/assets/70135188/10b076f3-17fa-43bf-a195-3ce7e536195c)


- Activity가 재생성 되어도 메모리에 남아 있습니다.

## ViewModel + 액티비티 유지 안함

- 개발자 옵션에서 액티비티 유지 안함 기능을 활성화하면 foreground상태가 아닌경우 ViewModel은 파괴됩니다.
    - 시스템에 의해 Activity 및 ViewModel 인스턴스가 GC 되므로 상태가 보존되지 않습니다.
- `ViewModel`은 영구적으로 상태를 보존하는 컴포넌트는 아닙니다.

## ViewModel 사용 시 유의사항

- ViewModel의 주요 목적은 구성이 변경되어도 상태를 유지하는 것입니다.
- 데이터를 영구 유지하련느 목적이면 로컬 디스크 또는 서버에 저장합니다.
- ActivityContext와 같은 객체를 참조하면 메모리 누수가 발생할 수 있습니다.
- ViewModel은 UI에 대한 의존성이 없어야 합니다.
- ViewModel을 다른 클래스, 함수에 전달하면 안됩니다.
    - ViewModel의 인스턴스는 시스템에서 관리하기 때문에 UI 시스템(Activity, Fragment, Composable)에 최대한 가깝게 하고, 다른 일반 클래스나 함수에 전달하지 않도록 주의해야 합니다.

## Hilt에서 ViewModel 인스턴스 얻기

```kotlin
@HiltViewModel
class FooViewModel @Inject constructor(
    val foo: Foo,
    val handle: SavedStateHandle
): ViewModel()

@AndroidEntryPoint
class MainActivity: AppCompatActivity {
    val viewModel: FooViewModel by viewModels()
}
```

- Hilt를 사용하면 ViewModel의 인스턴스화가 간단해집니다.
    - 매번 ViewModel.Factory코드를 생성하지 않아도 됩니다.
- Hilt에서 공통적으로 사용 가능한 `HiltViewModelFactory`를 제공해주기 때문입니다.
    - `by viewModels()` 로 호출이 가능합니다.
        - 이는 바이트코드 변조 과정으로 이루어 집니다.
- `@HiltViewModel` 애노테이션이 ViewModel 클래스에 마킹 되어야 Hilt 컴파일러에서 해당 애노테이션을 해석하고, 의존성 주입이 가능하게 됩니다.

## @HiltViewModel을 사용하는 이유

![스크린샷 2024-04-02 오후 11 36 24](https://github.com/jiwon2724/TIL/assets/70135188/e688e391-58f6-411e-94a8-c8837b242a0e)


- ViewModelStore에 동일한 클래스의 오브젝트들이 여러개 저장될 수 있기 때문입니다.

## ViewModelComponent

![스크린샷 2024-04-02 오후 11 39 46](https://github.com/jiwon2724/TIL/assets/70135188/cd1055fd-7283-40da-bf2c-deb60e6e9ffe)


- ViewModel의 경우는 독립된 `ViewModelComponent`로 분리되어 의존성 주입이 일어납니다.
- 액티비티보다 오래 살아남기 위해 `ActivityRetainedComponent` 하위에 위치합니다.
    - `ActivityRetainedComponent`는 구성변경이 일어나도 컴포넌트 인스턴스가 유지됩니다.
- `ViewModelComponent` 내에서 여러 뷰모델 인스턴스를 관리하기 위해 `Map` 멀티 바인딩 기법을 내부적으로 사용합니다.
    - 문자열을 Key로 가지고, ViewModel 클래스 인스턴스를 Value로 갖습니다.

## ViewModelComponent와 @ViewModelScoped

```kotlin
@ViewModelScoped
class Bar @Injtet constructor(..)

@Module
@InstallIn(ViewModelComponent::class)
object FooModule {
    @Provides
    fun provideFoo(bar: Bar): Foo {
        return Foo(bar)
    }
} 
```

- ViewModelComponent에서 의존성을 공유하고 싶다면 `@ViewModelScoped` 애노테이션을 사용할 수 있습니다.
