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

> Dagger가 Null 바인딩을 기본적으로 금지하기 때문에, Null을 리턴하는 것을 주의해야합니다.

## @Binds 바인딩

- 기존에 바인딩된 의존성이 있다면, 새로운 `Provider` 메서드를 만들지 않고 효율적으로 바인딩 할 수 있는 방법입니다.
