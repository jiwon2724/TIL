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

## Map 멀티 바인딩

- 동일한 타입의 의존성들을 Map형태로 관리합니다.
    - Key, Value로 관리하므로, 의존성을 관계를 맺을 Key가 반드시 필요합니다.

## Map 멀티 바인딩을 위한 기본 키

- @StringKey
- @IntKey
- @LongKey
- @ClassKey

## Map 멀티 바인딩 with @IntoMap

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object MyModule {
    @Provides
    @IntoMap @StringKey("foo")
    fun provideFooValue(): Long {
        return 100L
    }
		
    @Provides
    @IntoMap @ClassKey(Bar::class)
    fun provideBarValue(): String {
        return "value for bar"
    }
}
```

- `String` 타입의 Key, `Long` 타입의 Value를 갖습니다.
- `Class` 타입의 Key, `String` 타입의 Value를 갖습니다.

> 즉, Map<String, Long>과 Map<Class<*>, Long>이 Hilt Compoent에 바인딩 됩니다.
”foo” : 100L, Class<Bar> : “value for bar”
> 

## 멀티 바인딩 된 Map 주입

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity {
    @Inject
    lateinit var map: Map<String, Long>
		
    @Inject
    lateinit var map2: Map<Class<*>, String>
		
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        map1["foo"].toString() // 100
        map2[Bar::class.java] // value for Bar
    }
}
```

- `Map`의 제네릭 타입을 체크하고 주입이 되기 때문에, 제네릭 타입을 잘 신경써야 합니다.

## @MapKey

```kotlin
enum class MyEnum {
    ABC, DEF
}

@MapKey
annotation class MyEnumKey(val value: MyEnum)

@Module
@InstallIn(SingletonComponent::class)
object MyModule {
    @Provides
    @IntoMap
    @MyEnumKey(MyEnum.ABC)
    fun provideABCValue(): String {
        return "value for ABC"
    }
}

class MainActivity : ComponentActivity() {
    @Inject
    lateinit var map: Map<MyEnum, String>
		
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        map[MyEnum.ABC] // value for ABC
    }
}
```

- 해당 애노테이션으로 커스텀 키를 만들 수 있습니다.

## @Multibinds

```kotlin
@Module
@Install(SingletonComponent::class)
abstract class MyModuleA {
    @Multibinds
    abstract fun fooSet(): Set<Foo>
		
    @Multibinds
    abstract fun fooMap(): Map<String, Foo>
}
```

- 하나 이상 의존성을 생성하고, 멀티 바인딩 해야지만 Map, Set이 생성 되었지만, 추상적으로 빈 Map, Set을 생성하기 위해 멀티 바인딩을 구현하려면 `@Multibinds` 애노테이션을 사용하면 됩니다.
- `@Multibinds` 애노테이션을 사용하면 컴파일 타임에 의존성이 없더라도, 에러 없이 작업을 수행할 수 있습니다.
- 즉, 바인딩 여부가 확실하지 않은 경우 컴파일 타임에 빈 컬렉션을 제공합니다.
