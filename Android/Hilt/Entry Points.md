# Entry Points

- Hilt 의존성 주입이 어려운 코드에서 바인딩 된 의존성을 참조하는 방법입니다.
- Hilt 컴포넌트에 바인딩된 의존성들에 접근할 수 있는 진입접을 제공합니다.

## @AndroidEntryPoint vs @EntryPoint

- `@AndroidEntryPoint` 애노테이션은 안드로이드 컴포넌트(Activity, Fragment, Service)클래스에 사용되는 EntryPoint 애노테이션 입니다.
- `@EntryPoint` 애노테이션은 안드로이드 클래스 뿐만 아니라, 다른 클래스에서도 Hilt 컴포넌트에 접근할 수 있는 진입점을 제공합니다.


## Entry Points는 언제 사용하나요?

- Hilt를 사용하지 않는 라이브러리에서 의존성 주입이 어려울 때
- Hilt가 지원하지 않는 안드로이드 컴포넌트에 의존성 주입이 어려울 때
- Dynamic Feature Module에 의존성 주입이 필요한 경우

## EntryPoint 만들기

```kotlin
@EntryPoint
@InstallIn(SingletonComponent::class)
interface FooEntryPoint {
    fun getFoo(): Foo
}
```

- Hilt 컴포넌트 의존성에 접근하기 위해 `@EntryPoint` 인터페이스를 정의해야 합니다.
    - 인터페이스의 재정의 함수는 매개변수를 갖지 않아야 하고, 반드시 반환타입이 있어야 합니다.
- `@EntryPoint` 와 `@InstallIn` 을 마킹하면 컴파일 타임에 해당 엔트리 포인트(`FooEntryPoint`) 인터페이스를 확장한 Hilt가 컴포넌트를 생성하게 됩니다.
- `SingletonComponent`로 스코프를 한정했기 때문에 `FooEntryPoint` 의존성 제공 범위도 `SingletonComponent` 로 한정짓게 됩니다.
    - ex) Foo 의존성은 반드시 `SingletonComponent` 스코프 내에 바인딩 되어 있어야 합니다.

> SingletonComponent에 바인딩된 Foo의존성을 접근하기 위한 EntryPoint 준비가 끝났습니다.
> 

## EntryPoint 접근하기

```kotlin
val fooEntryPoint: FooEntryPoint = EntryPoints.get(applicationContext, FooEntryPoint::class.java)
val foo: Foo = fooEntryPoint.getFoo()
```

- 위 코드에선 `SingletonComponent` 스코프로 지정되었으므로, `applicationContext`를 사용해야합니다.

## EntryPointAccessors

```kotlin
val fooEntryPoint = EntryPointAccessors.fromApplication(context, FooEntryPoint::class.java)
val foo: Foo = fooEntryPoint.getFoo()
```

- EntryPoints.get에 들어가는 첫 번째 인자는 Object 타입입니다.
- 어떤 인자를 전달해야하는지 모호하기 때문에 `EntryPointAccessors` 클래스는 명확하게 안드로이드 클래스를 제공할 수 있습니다.
    - 이는 타입에 안전하게 접근할 수 있습니다.
    - 내부적으론 `EntryPoints` 클래스에 접근합니다.

## EntryPointAccessors 메서드

- fromApplication
- fromActivty
- fromFragment
- fromView
