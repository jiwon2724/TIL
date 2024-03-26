# Hilt 바인딩 기법

## 바인딩

- 의존성을 컴포넌트에 추가 하는 행위 또는 그 의존성을 일컫는 용어

## 바인딩의 종류

1. `@Inject`를 활용한 생성자 바인딩
2. `@Provides`를 활용한 바인딩
3. `@Binds`를 활용한 바인딩
4. `@BindsOptionalOf`를 활용한 바인딩
5. `@BindsInstance`를 활용한 바인딩

## 생성자 바인딩 with @Inject

```kotlin
class Foo @Inject construct()
```

- `@Inject`는 의존성을 요청할 때도 사용되지만, 생성자에 마킹하는 경우는 해당 의존성과 함께 컴포넌트에 바인딩 됩니다.

## @Provides 바인딩

```kotlin
@Module
@IntallIn(..)
object MyModule {
    @Provides
    fun provideFoo(..): Foo { .. }
}
```

- 모듈 클래스에서 `@Provides` 애노테이션과 함께 메서드를 선언할 수 있습니다.
- 메서드 파라미터에 이미 오브젝트 그래프에 바인딩된 의존성을 요청할 수 있습니다.
- 메서드의 리턴 타입이 그래프에 바인딩 되는 방식입니다.

> Dagger가 Null 바인딩을 기본적으로 금지하기 때믄에, Null을 리턴하는 것을 주의해야합니다.

## @Binds 바인딩, 컴파일 타임 Missing Bidning오류

```kotlin
interface Engine
class GasolineEngine @Inject constructor() : Engine
```

- 기존에 바인딩된 의존성이 있다면, 새로운 `Provider` 메서드를 만들지 않고 효율적으로 바인딩 할 수 있는 방법입니다.
- 가솔린 엔진이 HiltComponent에 바인딩 됩니다.
- 위 코드에선 가솔린 엔진을 엔진으로 캐스팅이 가능합니다.

<img width="937" alt="스크린샷 2024-03-23 오후 3 55 56" src="https://github.com/jiwon2724/TIL/assets/70135188/e3e923b2-c24d-4221-9da7-7d9794e1be81">


- 클라이언트는 가솔린 엔진이 아닌 엔진 타입을 컴포넌트에 요청했지만, 컴포넌트에 바인딩된 타입은 가솔린 엔진입니다.
- 위 경우 힐트는 컴파일 타임에 해당 의존성을 찾을수 없다면서 `Missing Binding`에러 출력과 컴파일을 중단시킵니다.
- 가솔린 엔진을 엔진으로 제공하는 `Provider` 메서드를 만들 수 있지만, 더 좋은 방법은 `@Binds` 애노테이션을 사용하는 것입니다.

## @Binds 바인딩

```kotlin
@Module
@InstallIn(..)
abstract class EngineModule { 
    @Binds
    abstract fun bindEngine(engine: GasolineEngine): Engine
}
```

- 추상 메서드이기 때문에, 클래스도 추상 클래스이어야 합니다.
- 메서드의 파라미터는 바인딩 되었던 의존성을 명시하면 됩니다.
- 반환 타입은 캐스팅 가능한 상위 타입을 설정합니다.
    - 가솔린 엔진은 엔진으로 캐스팅이 가능합니다.

<img width="679" alt="스크린샷 2024-03-23 오후 4 24 09" src="https://github.com/jiwon2724/TIL/assets/70135188/84abad2c-afe4-4db0-86f1-dd9da8561b02">

- 컴포넌트 내부에서는 이미 바인딩된 가솔린 엔진을 엔진으로 재바인딩을 하게 됩니다.

## @Binds 활용 예시, 선택적 모듈 설치

<img width="946" alt="스크린샷 2024-03-23 오후 4 38 18" src="https://github.com/jiwon2724/TIL/assets/70135188/0c8b45f7-241d-421f-a31c-dccec0fe451b">

- 클라이언트(Car)의 코드를 수정하지 않는 것이 DI 패턴를 사용하는 이점입니다.
- 자동차라는 객체가 있고 어떤 엔진을 주입 하는지에 따라서 자동차 엔진의 타입이 결정됩니다.
    - 즉, 어떤 의존성을 바인딩 시킬 모듈이 컴포넌트에 바인딩 되는 여부에 따라서 자동차의 상태나 동작이 결정됩니다.

## @Binds 제약 조건

- `@Binds`는 반드시 모듈 내의 `abstract` 메서드에 추가해야 합니다.
- `@Binds` 메서드는 반드시 파라미터 1개만을 가집니다.
- 파라미터 타입이 반환타입의 서브타입이어야 합니다.
- Scope 및 Qualifier 애노테이션과 함께 사용할 수 있습니다.

## @BindsOptionalOf

![스크린샷 2024-03-26 오후 10 57 21](https://github.com/jiwon2724/TIL/assets/70135188/689256b5-0ea9-49bc-a0de-cbf981808be8)


- `@BindsOptionalOf` 애노테이션은 오브젝트 그래프에 바인딩 되어있을 수도 있고, 없을 수도 있습니다.
    - 즉, 바인딩 되어 있지 않을 가능성이 있는 의존성을 요청할 때(옵셔널 바인딩)
- 옵셔널 바인딩을 사용하면, 바인딩 되지 않았더라도, 컴파일이 통과하게 됩니다.
    - 위 이미지에선 클라이언트가 `Optional<Foo>`를 요청한다면, `Foo`의 바인딩 여부와 관계없이 Hilt의 컴파일 과정을 통과합니다.

## @BindsOptionalOf 바인딩

```kotlin
@Module
@IntallIn(SingletonComponent::class)
abstract class FooModule {
    @BindsOptionalOf
    abstract fun optionalFoo(): Foo
}
```

- 옵셔널 바인딩 메서드의 반환 타입으로 옵셔널 바인딩 하고자하는 타입을 선언하면 됩니다.

## Optional<T> 요청

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    @Inject
    lateinit var optionalFoo: Optional<Foo>
}

class Bar @Inject constructor(optionalFoo: Optional<Foo>)

@Provides
fun provideString(optionalFoo: Optional<Foo>): String { .. }
```

- 옵셔널 바인딩도 마찬가지로 바인딩된 의존성들은 똑같이 의존성 주입을 요청하고 주입을 받을 수 있습니다.

## Optional<T> 주요 메서드

- `isPresent()`
    - 바인딩 된 경우 true를 반환합니다.
- `get()`
    - 바인딩 된 의존성 T를 반환합니다
    - 바인딩 되지 않은 경우 예외를 던집니다.
    - orElse 류의 메서드 호출로 안전하게 접근할 수도 있습니다.

## @BindsOptionalOf 제약조건

- `@BindsOptionalOf` 는 반드시 모듈 내의 `abstract` 메서드에 추가해야합니다.
- `@BindsOptionalOf` 메서드는 반드시 void 타입을 반환하면 안됩니다.
    - 반환 타입이 반드시 있어야 합니다.
- `@BindsOptionalOf` 메서드는 파라미터를 가질 수 없습니다.
- 생성자 바인딩 된 의존성은 항상 `present` 상태이므로, 이 경우 해당 의존성은 옵셔널 바인딩이 불가능합니다.
- `Optional<Provider<T>>`, `Optional<Lazy<T>>`, `Optional<Provider<Lazy<T>>>` 형태로 주입이 가능합니다.
