# Lazy와 Provider

## Lazy 주입

```kotlin
@AndroidEntryPoint
class MainActivity: ComponentActivity() {
    @Inject
    lateinit var fooLazy: Lazy<Foo>
		
    override fun onCreate(saveInstanceState: Bundle?) {
        super.onCreate(saveInstanceState)
				
        // get() 호출 시 인스턴스화, Foo객체가 생성됩니다.
        val foo1 = fooLazy.get()
    }
}
```

- Lazy 주입은 일반적인 주입과 조금 다릅니다.
- 즉, 늦은 인스턴스 생성을 의미합니다.
- `get()` 메서드를 사용할 때 Foo 객체가 생성됩니다.

> Lazy를 사용하면 컴파일 시 HiltComponent 바인딩된 상태는 아닙니다. get 메서드를 통해서 의존성을 가져옵니다. 초기화를 늦추고 특정 시점에 인스턴스를 생성할 수 있습니다.
> 

## Lazy 주입 특징

- `Lazy<T>` 의`get()` 메서드를 호출할 때 `T`를 반환합니다. `get()` 호출 시점에 `T`가 인스턴스화 됩니다.
- `Lazy<T>` 의`get()` 호출이후 다시 `get()` 을 호출하면 캐싱 된(동일한) `T` 인스턴스를 얻습니다.
- `T` 바인딩에 Scope가 지정되어 있다면, 각 `Lazy<T>` 요청에 대한 동일한 `Lazy<T>` 인스턴스가 주입됩니다.
    - 해당 컴포넌트의 생명주기에 따라 동일한 의존성을 반환합니다.

```kotlin

// CASE 1 :
class Foo @Inject constructor()

@AndroidEntryPoint
class MainActivity: ComponentActivity() {

    @Inject lateinit var lazyFoo1: Lazy<Foo>
    @Inject lateinit var lazyFoo2: Lazy<Foo>
		
    override fun onCreate(saveInstanceState: Bundle?) {
        super.onCreate(saveInstanceState)
        assert(lazyFoo1 !== lazyFoo2) // true
    }
}

// CASE 2 :
@Singleton
class Foo @Inject constructor()

@AndroidEntryPoint
class MainActivity: ComponentActivity() {

    @Inject lateinit var lazyFoo1: Lazy<Foo>
    @Inject lateinit var lazyFoo2: Lazy<Foo>
		
    override fun onCreate(saveInstanceState: Bundle?) {
        super.onCreate(saveInstanceState)
        assert(lazyFoo1 !== lazyFoo2) // false
    }
}
```

- 특정시점에 바인딩 인스턴스화 할 때 사용하면 좋습니다.
- 인스턴스 생성에 비용이 큰 경우 사용하면 좋습니다.
    - 선택적으로 인스턴스화를 해야하는 경우

> lateinit을 사용하여 Lazy<T> 타입을 사용할 때 isInitialized 프로퍼티를 검사하는 것은 Lazy<T> 인스턴스 자체의 초기화 여부를 확인하는 것이 아니라, Lazy<T>에 의해 관리되는 값(T 타입의 인스턴스)의 초기화 여부를 확인하는 것입니다.
> 

## Provider

```kotlin
@AndroidEntryPoint
class MainActivity: ComponentActivity() {
    @Inject
    lateinit var fooProvider: Provider<Foo>
		
    override fun onCreate(saveInstanceState: Bundle?) {
        super.onCreate(saveInstanceState)
				
        // 매 get() 호출마다 인스턴스 생성
        val foo1 = fooProvider.get()
    }
}
```

- Lazy와 마찬가지로 주입 받고자하는 대상 의존성을 제네릭 타입으로 갖는 `Wrapper Provider`를 주입 받는 방식입니다.

> Lazy는 초기에 get() 메서드를 호출하면 값이 캐싱되어 다음 get() 메서드 호출 시 캐싱된 값으로 반환되지만, Provider는 get() 메서드를 호출 시 매번 새로운 인스턴스가 생성됩니다.
> 

## Provider 주입 특징

- Provider<T>의 get() 메서드를 호출할 때마다 새로운 인스턴스 T를 반환합니다.
- T 바인딩에 Scope가 지정되어 있다면, Provider<T>의 get() 메서드를 호출할 때 동일한 인스턴스 T를 반환합니다.
- T 바인딩에 Scope가 지정되어 있다면, 각 Provider<T> 요청에 대한 동일한 Provider<T> 인스턴스가 주입됩니다.
- 하나의 Provider<T>로 여러 T 인스턴스를 원할 때 생성할 수 있다. (Builder, Factory 패턴과 유사)

```kotlin

// CASE 1 :
class Foo @Inject constructor()

@AndroidEntryPoint
class MainActivity: ComponentActivity() {
    @Inject lateinit var providerFoo1: Provider<Foo>
    @Inject lateinit var providerFoo2: Provider<Foo>
		
    override fun onCreate(saveInstanceState: Bundle?) {
        super.onCreate(saveInstanceState)
				
        val foo1 = providerFoo1.get()
        val foo2 = providerFoo1.get()

        assert(foo1 !== foo2) // true
		}
}

// CASE 2 :
@Singleton
class Foo @Inject constructor()

@AndroidEntryPoint
class MainActivity: ComponentActivity() {

    @Inject lateinit var providerFoo1: Provider<Foo>
    @Inject lateinit var providerFoo2: Provider<Foo>
		
    override fun onCreate(saveInstanceState: Bundle?) {
        super.onCreate(saveInstanceState)
				
        val foo1 = providerFoo1.get()
        val foo2 = providerFoo1.get()

        assert(foo1 !== foo2) // false
    }
}
```

- `Provider` 가 스코핑 되어있다면 `get()`을 호출해도 새로운 인스턴스를 생성하는게 아니라 이미 스코핑된 인스턴스를 반환합니다.
    - 즉, 여러번 호출해도 인스턴스는 같습니다.
 
> Lazy와 Provider의 공통점은 인스턴스화 시점을 늦출 수 있습니다.
