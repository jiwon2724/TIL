# @Qualifier 활용

## Hilt는 Type으로 의존성을 구분한다

- 의존성이 `HiltComponent`에 바인딩 및 주입 될 때, Hilt는 의존성을 식별할 수 있는 식별자로 클래스의 패키지명을 포함하고 있는 `캐노니컬 네임(CNAME)` 을 식별자로 활용하게 됩니다.
    - 다른 용어로는 `시그니처`라고 부릅니다.
    - ex) `ArrayList`의 캐노니컬 네임은 `java.util.ArrayList`

> 클라이언트가 의존성 주입을 요청할 때 힐트가 이를 정확하게 구분하고 수행 할 수 있는 이유는 애노테이션이 마킹된 위치에 명확하게 해당 의존성 타입이 명시되어있기 때문입니다.
> 

## 중복 바인딩

<img width="564" alt="스크린샷 2024-03-17 오후 3 27 43" src="https://github.com/jiwon2724/TIL/assets/70135188/3618d41a-0494-45c8-93a6-fd68bc09a431">


- 동일한 타입이 중복되게 바인딩 될 수 있습니다.
- 위 같은 경우에 `HiltComponent` 는 어떤 의존성을 주입해야하는지 알 수 없습니다.

> Hilt는 컴파일 타임에 오브젝트 그래프를 점검하고, 중복 바인딩이 문제가 있는 경우에 컴파일을 중단시킵니다.
> 

## @Qualifier

- 중복 바인딩을 사용하고 싶은 경우가 있습니다.
    - 중복 바인딩을 유지하면서, 사용하고 싶은 바인딩을 구분해서 사용하는 경우
- `@Qualifier` 애노테이션은 중복 바인딩된 의존성 타입이 컴포넌트에 존재하더라도 컴포넌트는 `@Qualifier` 를 통해서 구분할 수 있게 됩니다.

## 커스텀 Qualifier 선언

```kotlin
@Qualifier
annotation class CustomQualifier
```

- `@Qualifier` 를 직접적으로 사용하지는 못합니다.
- `@Qualifier` 를 애노테이션을 갖는 새로운 애노테이션을 정의해야합니다.
    - 위 코드 참조

> annotation class 생성자 파라미터에 프로퍼티를 지정해 줄 수 있습니다.
> 

## 커스텀 Qualifier 활용

```kotlin
@Module
@InstallIn(SingletonComponent::class)
// 모듈이 설치되는 위치는 SingletonComponent입니다.
object AppModule {
    @CustomQualifier
    @Provides
    fun provideFoo1(): Foo { // Foo를 바인딩
        return Foo()
    }

    @Provides
    fun provideFoo2(): Foo { // Foo를 바인딩
        return Foo()
    }
}

@AndroidEntryPoint
class MainActivity : ComponenetActivity() {
		@CustomQualifier
		@Inject
		lateinit var foo: Foo // provideFoo1이 주입됩니다.
}
```

- `Foo` 를 리턴하는 provide 메서드가 두 개 선언 돼서 중복 바인딩이 되어도 `@CustomQualifier` 애노테이션이 마킹 되어 있으므로 컴파일 타임에 오브젝트 그래프를 생성할 때 두 메서드를 구분하게 됩니다.

## @Named 활용하기

```kotlin
@Module
@InstallIn(SingletonComponent::class) 
// 모듈이 설치되는 위치는 SingletonComponent입니다.
object AppModule {
    @Named("Foo1")
    @Provides
    fun provideFoo1(): Foo { // Foo를 바인딩
        return Foo()
    }

    @Provides
    fun provideFoo2(): Foo { // Foo를 바인딩
        return Foo()
    }
}
```

- `@Named` 애노테이션을 사용하면 중복 바인딩을 더 간단하게 만들 수 있습니다.
- `@Named` 애노테이션 파라미터에 문자열을 지정하면 됩니다.
    - 해당 문자열로 `HiltComponent`는 의존성을 구분하게 됩니다.
