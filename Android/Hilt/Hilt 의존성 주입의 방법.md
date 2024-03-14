# Hilt 의존성 주입의 방법

## 의존성 주입

![스크린샷 2024-03-12 오후 10 41 04](https://github.com/jiwon2724/TIL/assets/70135188/6ef13260-a12c-46e6-a9b6-a510dc651757)


- Hilt와 관계 없이 의존성 주입 패턴을 보면 의존성을 요청하는 대상을 `Client`라 부릅니다.
- 의존성 관리를 제공하는 주체를 `Container`라고 부릅니다.

![스크린샷 2024-03-12 오후 10 41 19](https://github.com/jiwon2724/TIL/assets/70135188/0a088163-644d-4513-bee2-845cf614e1a6)


- Hilt를 대입해서 보면 `Client` 는 Android Class 대상이 되고, 이는 `Application`, `Activity`, `Service`, `Fragment`, `Broadcast Receiver`, `ViewModel`등 을 의미합니다.
- `Container` , `Injector` 라고 부르는 주체를 `Hilt Component`라고 합니다.
- `Hilt Component` 는 의존성의 생명주기를 관리하고, 의존성을 제공하는 역할을 합니다.
    - Hilt 어노테이션 프로세서는 컴파일 타임에 애노테이션 프로세싱을 통해 의존성 그래프를 검사하여 문제가 없는지 확인하고, 의존성 주입을 위한 코드를 생성합니다.
    - 위 과정으로 생성된 코드들이 런타임에 실행되면서 의존성 주입을 가능하게 합니다.

## Hilt 의존성 바인딩

![스크린샷 2024-03-12 오후 10 52 00](https://github.com/jiwon2724/TIL/assets/70135188/dd1ae86a-cc2a-45e5-9cfc-9f18e7d7f2fe)


- 컴포넌트에 의존성을 추가하고 관리하는 행위를 `바인딩`이라고 부릅니다.
- 의존성을 추가하는 행위 자체도 `바인딩`이라고 부르고, 하나의 개별 의존성 자체도 `바인딩`이라고 부릅니다.
- 상황과 문맥에 맞게 `바인딩`이라는 용어를 사용하는게 좋습니다.

## Hilt 의존성 바인딩 with Module

![스크린샷 2024-03-12 오후 10 52 39](https://github.com/jiwon2724/TIL/assets/70135188/c54be4b4-f4df-40ca-ae4b-aaed609c4b30)


- `Hilt Component`에 바인딩 하는 방법은 여러가지 있지만 크게 두 가지가 있습니다.
    1. `@Provides` 애노테이션 사용
    2. 생성자에 `@Inject` 애노테이션을 사용하여 오브젝트 그래프에 바인딩

![스크린샷 2024-03-12 오후 10 57 25](https://github.com/jiwon2724/TIL/assets/70135188/7d4de06c-1748-4c55-b6a8-7b031b319ad2)


> Hilt 의존성 주입의 추상적인 전체적인 흐름은 위 오브젝트 그래프와 같이 표현할 수 있습니다.
> 
- 위 그래프에 컴파일 타임에 결함이나 오류를 체크하는 것이 Hilt의 가장 큰 장점이라고 할 수 있습니다.

## 의존성 주입의 종류 with @Inject

> 의존성 주입은 `@Inject` 애노테이션을 사용하는 것으로 시작이 됩니다.
> 
- `@Inject` 애노테이션을 마킹할 수 있는 곳은 다음과 같습니다.
    1. 생성자 주입
    2. 필드 주입
    3. 메서드 주입

## 생성자 주입

```kotlin
class Foo @Inject construct(bar: Bar) { ... }
```

- 의존성 주입시 가장 보편적으로 사용하는 패턴입니다.
- 클래스 생성자 앞에 `@Inject` 애노테이션을 마킹하면 됩니다.
- `Bar`라는 의존성을 Hilt 컴포넌트에 요청하고, 런타임에 주입을 받습니다.
    - 생성자 파라미터로 주입을 받았기 때문에 `Foo`라는 객체를 인스턴스화 할 때 부터 해당 의존성을 참조할 수 있습니다.

## 필드 주입

```kotlin
@AndroidEntryPoint
class AndroidClass : ... {
    @Inject lateinit var bar: Bar
}
```

- 일반적으로 안드로이드 클래스에서는 생성자 주입을 할 수 없기 때문에, 필드 주입을 많이 활용합니다.
    - 안드로이드 클래스는 `Activity`, `Fragment`, `Service` 등을 의미합니다.
- 필드 주입이나 메서드 주입 시 `@AndroidEntryPoint` 애노테이션 마킹은 필수입니다.
- 필드 주입을 하는 경우엔 안드로이드 클래스의 필드에다 `@Inject` 애노테이션을 마킹하면 됩니다.
    - 필드는 안드로이드 클래스 내부의 코드 블록 안쪽을 의미합니다.
- 안드로이드 클래스의 시스템이 특정 시점에 직접 인스턴스화 합니다.
    - 개발자가 직접 인스턴스화 하지 않습니다.
- `lateinit`을 사용하여 주입받는 것이 가장 일반적인 방식입니다.

## 메서드 주입

```kotlin
@AndroidEntryPoint
class AndroidClass : ... {
    @Inject
    fun setBar(bar: Bar) { ... }
}
```

- 자주 활용되는 방식은 아니지만, 일부 케이스에서 유용하게 사용될 수 있습니다.
- 메서드의 파라미터가 요청할 의존성 입니다.
    - 위 코드에선 `Bar` 객체 입니다.
- 해당 의존성이 각 안드로이드 클래스별 의존성 주입 시점에 주입이 됩니다.
    - ex) `Activity`나 `Service`는 `onCreate`시점에 의존성 주입이 일어납니다.
    - ex) `Fragment`는 `onAttatch`시점에 의존성 주입이 일어납니다.

> 안드로이드 클래스에 의존성 주입을 하는 경우에 필드 주입, 메서드 주입 중에 필드 주입이 먼저 주입이 실행됩니다. 필드 주입 이후 메서드 주입이 일어납니다.
> 

## 생성자 주입과 바인딩을 동시에

```kotlin
class Foo @Inject construct(bar: Bar) { ... }
```
![스크린샷 2024-03-13 오후 11 32 42](https://github.com/jiwon2724/TIL/assets/70135188/75bf1aab-6467-4bb0-969d-49b09a020b90)

- 생성자에 `@Inject` 을 마킹하는 경우 컴포넌트로부터 파라미터(Bar)를 주입 받기도 하지만, 주입 받음과 동시에 주입 받는 대상 즉, `Foo` 클래스도 컴포넌트에 바인딩 됩니다.
- 어딘가에서 `Foo`의존성을 요청하면, 주입이 가능해집니다.
- 즉, @Inject 애노테이션은 하나의 마킹으로 두 가지 행위를 동시에 하고있습니다.

## Hilt 의존성 주입 예제

> 생성자 주입은 주입과 동시에 대상 자체가 바인딩 됩니다.
> 

## 필드 주입

```kotlin
// 컴파일 타임에 의존성 바인딩
// 주입과 동시에 바인딩됩니다. Foo class는 생성자 매개변수가 없으므로, Foo만 바인딩 됩니다.
class Foo @Inject constructor() { .. } 

@AndroidEntryPoint
class MainActivty : ...() {
    @Inject lateinit var foo: Foo // Foo 의존성 요청
			
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)
        assert(this::foo.isInitialized)
    }
}
```

- 컴파일 타임에 컴포넌트에 `Foo`라는 의존성이 바인딩 됩니다.

## 생성자 주입

```kotlin
class Foo @Inject constructor(val bar: Bar) { }
class Bar @Inject constructor() { }

@AndroidEntryPoint
class MainActivty : ...() {
    @Inject lateinit var foo: Foo
    @Inject lateinit var bar: Bar
			
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)
        assert(this::foo.isInitialized)
        assert(this::bar.isInitialized)
        assert(foo.bar != null)
    }
}
```

- 컴파일 타임에 `Bar` 의존성은 Foo 클래스의 생성자인 `bar`에 바인딩 됩니다.
- `@Inject lateinit var bar: Bar`
    - `Bar` 의존성을 요청합니다.

## 메서드 주입

```kotlin
class Foo @Inject constructor(val bar: Bar) { } 
class Bar @Inject constructor() { }

@AndroidEntryPoint
class MainActivty : ...() {
    lateinit var foo: Foo

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)
        assert(this::foo.isInitialized)
    }
    
    @Inject
    fun injectFoo(foo: Foo) {
        this.foo = foo
    }
}
```

- `super.onCreate` 호출 이후에 의존성 주입이 `injectFoo` 에서 발생하게 됩니다.

## 요약

- @Inject 애노테이션으로 의존성을 요청할 수 있다.
- 의존성 주입은 크게 생성자 주입, 필드 주입, 메서드 주입이 있다.
- 생성자 주입의 경우 주입과 동시에 주입받는 대상 자체가 의존성 그래프에 바인딩 된다.
