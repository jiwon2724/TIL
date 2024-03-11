# Hilt 모듈과 바인딩

## 모듈

- 프로그램을 구성하는 구성요소
- 관련된 데이터와 함수를 하나로 묶은 단위

## 컴포넌트와 모듈

![스크린샷 2024-03-11 오후 5 44 55](https://github.com/jiwon2724/TIL/assets/70135188/8c7b344c-1557-42c9-bac6-c1c361b45352)


> 컴파일 타임시 Hilt Component에 모듈을 설치하는 방식으로 의존성 주입의 설정(Setting)이 일어납니다.
> 

## 모듈이 설치되는 과정

- 모듈은 다음 과정을 거쳐 컴파일 타임에 컴포넌트에 설치됩니다.
1. Annotation Processor를 통해 `@Module` 애노테이션이 마킹된 클래스를 스캔합니다.
2. 해당 클래스에 `@InstallIn` 애노테이션이 있는지 확인 후 설치될 컴포넌트를 탐색합니다.
    1. `@InstallIn은` 해당 모듈이 설치될 컴포넌트의 경로가 함께 포함되어 있습니다.
3. 해당 컴포넌트에 설치됩니다.

## @InstallIn 사용하기

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object FooModule {
    @Provides
    fun provideBar(): Bar { ... }
}
```

- `@Module`이 마킹된 클래스에 `@InstallIn` 애노테이션을 추가할 수 있습니다.
- `@InstallIn`의 속성으로 컴포넌트 클래스를 명시하면 해당 컴포넌트에 `FooModule` 이 가지고 있는 의존성들이 바인딩 됩니다.
    - 위 코드에서 해당 컴포넌트는 `SingletonComponent`입니다.
- `provideBar` 함수를 통해 `Bar`라는 의존성이 `SingletonComponent` 에 바인딩 됩니다.

> @Module 애노테이션만 존재하고, @InstallIn 애노테이션이 존재하지 않을 경우 컴파일 타임에 이 부분을 체크해여 에러가 발생합니다. 즉, @Module과 @InstallIn은 같이 존재해야합니다.
> 

```kotlin
@Module
@InstallIn(SingletonComponent::class)
internal object FooModule {
    @Singleton
    @Provides
    fun provideBar(): Bar { ... }
}
```

- 이전 코드에선 `Bar` 의존성을 요청할 때 마다 `provideBar` 함수를 호출했습니다.
    - 즉, 새로운 `Bar` 인스턴스를 반환했습니다.
- 위 코드에선 `provideBar` 함수에 `@Singleton` 애노테이션을 추가했습니다.
    - 이렇게 되면 `Bar` 인스턴스를 컴포넌트 내부에 저장하기 때문에 `provideBar` 함수 호출은 단 한 번만 발생합니다.
    - `Bar` 인스턴스는 `SingletonComponent` 와 생명주기를 같이하게 됩니다.
        - 즉, 애플리케이션이 종료되기 전 까진 메모리에 남아있습니다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object FooModule {
    @Provides
    fun provideBar(app: Application): Bar { ... }
}
```

- 모듈 클래스에서 provide 하는 함수(`provideBar`)는 모듈 클래스 자기 자신이 설치된 컴포넌트 바인딩에 접근하는 것이 가능합니다.
    - `SingletonComponent`는 기본 바인딩으로 `Application`을 제공합니다.
    - `provideBar`함수의 파라미터로 `Application` 을 선언하면, `Application` 인스턴스를 주입을 받고, 참조를 하는 것이 가능합니다.

## 여러 컴포넌트에 모듈 설치

```kotlin
ex)
@InstallIn(ViewComponent::class, ViewWithFragmentComponent::class)
```

- 중복된 코드 작성을 피하고자 모듈을 여러 컴포넌트에 설치하고 싶은 경우가 있습니다.
    - 제약적이지만, 여러 컴포넌트에 동일한 모듈을 설치하는 것이 가능합니다.
- 위 코드 처럼 컴포넌트 클래스를 모두 포함시키면 됩니다.
- 하지만 이 경우에는 반드시 지켜야하는 규칙 3가지가 있습니다.
    1. 프로바이드 함수의 파라미터로 요청하는 바인딩은 모두 동일한 스코프에 있어야 합니다.
        1. `ViewComponent`, `ViewWithFragmentComponent`는 동일한 `ViewScope` 애노테이션을 사용합니다.
    2. 프로바이드 함수에서 요청하는 의존성을 해당 컴포넌트에서 찾을 수 있어야 합니다.
    3. 하위 및 상위 컴포넌트 계층간에는 동일한 모듈을 설치 할 수 없습니다. 
        1. 중복된 타입의 바인딩이 내부에서 발생하기 때문에, 상위 컴포넌트에만 모듈을 설치하고 하위 모듈 및 컴포넌트에서는 이러한 바인딩에 대해서 접근할 수 있습니다.

## Nullable 바인딩은 지양

1. Hilt는 자바코드에서 null 바인딩을 금하고 있습니다.
    1. null을 바인딩하는 경우엔 런타임 에러가 발생합니다. 
2. 자바코드에서 의존성 바인딩 및 요청 시 `@Nullable` 애노테이션을 사용해서 우회할 수 있습니다.
    1. 추천하는 방법은 아닙니다.
3. 코틀린에서는 문법자체적으로 null assign을 금하고 있으나, Nullable 타입이 존재합니다.
    1. 타입 뒤에 `?` 를 통해서 Nullable하게 만들며, 힐트에서 Nullable 타입으로 바인딩과 주입이 가능합니다.
        1. 바인딩과 주입이 가능하더라도, null safety한 코드를 작성하기 위해 Nullable타입은 바인딩 하는 것은 좋지 않습니다.
