# Hilt의 표준 컴포넌트

![hilt-hierarchy](https://github.com/jiwon2724/TIL/assets/70135188/cef8ddee-f527-40d6-b9fe-a7573fb9977d)


- 컴포넌트 계층의 각 어노테이션들은 해당 컴포넌트의 라이프 사이클에 대한 의존성 범위를 지정하는데 사용됨.
- 컴포넌트간 화살표 방향은 하위 컴포넌트를 가르킴.
    - ActivityRetainedComponent는 SingletonComponent의 하위 컴포넌트임.
    - 하위 컴포넌트는 상위 컴포넌트 바인딩에 접근할 수 있음.
        - 역방향은 안됨.

## 컴포넌트의 생명주기

| Component | Scope | Created at | Destroyed at |
| --- | --- | --- | --- |
| SIngletomComponent | @Singlton | Application#onCreate() | Application destroyed |
| ActivityRetainedComponent | @ActivityRetainedScoped | Activity#onCreate() | Activity#onDestroy() |
| ViewModelComponent | @ViewModelScoped | ViewModel created | ViewModel destroy |
| ActivityComponent | @ActivityScoped | Activity#onCreate() | Activity#onDestroy() |
| FragmentComponent | @FragmentScoped | Fragment#onAttach() | Fragment#onDestroy() |
| ViewComponent | @ViewScoped | View#super() | View destroyed |
| ViewWithFragmentComponent | @ViewScoped | View#super() | View destroyed |
| ServiceComponent | @ServiceScoped | Service#onCreate() | Service#onDestry() |
- Hilt 컴포넌트가 언제 생성되고, 소멸되는지 위 도표로 알 수 있음.
- 컴포넌트는 Created at 부분에서 생성되어 주입이 일어나고, Destroyed at에 컴포넌트는 소멸됨.

## Scope vs Unscope

```kotlin
// 이 바인딩은 매 요청시 새로운 인스턴스를 생성함.
class UnscopedBinding @Inject constructor() { ... }

// 이 바인딩은 스코프 애노테이션이 있으므로
// 해당 Hilt 컴포넌트의 수명동안은 매 요청에 동일 인스턴스 반환을 보장함.
@FragmentScoped
class UnscopedBinding @Inject constructor() { ... }
```

## 모듈에서 Scope 지정하기

```kotlin
@Module
@InstallIn(FragmentCompnent::class)
object FooModule {
    @Provides
    fun provideUnscopedBidning() = UnscopedBinding()

    @Provides
    @FragmentScoped
    fun provideUnscopedBidning() = ScopedBinding()
}
```

## Scope Annotation은 언제 사용해야할까?

1. 어떤 의존성을 인스턴스화하는 비용이 클 때
2. 동일한 인스턴스 반환을 원할 때
3. 특정 인스턴스를 공유 하고 싶을 때
