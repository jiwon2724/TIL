## 멀티 바인딩

- 여러 의존성을 하나의 컬렉션으로 관리
    - 이는 컬렉션 자체를 컴포넌트에 바인딩하는 기법입니다.
        - `Set`, `Map`을 지원합니다.

## Set 멀티 바인딩

- 동일한 형태의 의존성들을 Set형태로 관리합니다.

## @Set 멀티 바인딩 with @IntoSet

![멀티바인딩1](https://github.com/jiwon2724/TIL/assets/70135188/a776c981-392e-43b4-8338-06db1a2baca2)


```kotlin
@Module
@InstallIn(SingletonComponent::class)
class ModuleA {
    @Provides
    @IntoSet
    fun provideOneString(): String {
        return "ABC"
    }
}
```

- `@IntoSet` 애노테이션을 사용하면 컴파일타임에 컴포넌트 내에 Set을 하나 생성하고, Set의 제네릭 타입인 의존성을 추가할 수 있는 환경을 만들어줍니다.
- `@IntoSet` 애노테이션을 추가하게 되면 provide 메서드의 타입이 단독으로 바인딩 되지 않습니다.
    - 위 코드에선 String을 단독으로 사용할 수 없고, String을 요청하게 되면 Missing Binding에러가 발생합니다.

## Set 멀티 바인딩 with @ElementsIntoSet

![스크린샷 2024-03-29 오후 4 21 20](https://github.com/jiwon2724/TIL/assets/70135188/7a128ab7-6bde-489f-81e1-013b36154e39)


```kotlin
@Module
@InstallIn(SingletonComponent::class)
class ModuleB {
    @Provides
    @ElementsIntoSet
    fun provideOneString(): Set<String> {
        return listOf("DEF", "GHI").toSet()
    }
}
```

- `@IntoSet` 을 통하여 의존성을 하나씩 멀티 바인딩 하는 방식과 달리 `@ElementsIntoSet` 애노테이션은Set 타입의 요소들을 멀티바인딩 하는 방법입니다.
- `ModuleA`의 “ABC”의 문자열을 포함해서 Set에서 총 3가지의 요소가 바인딩 되었습니다.

## 멀티 바인딩 된 Set 주입

```kotlin
class Bar @Inejct constructor(val strings: Set<String>) {
    init {
        assert(string.contains("ABC"))
        assert(string.contains("DEF"))
        assert(string.contains("GHI"))
    }
}
```
