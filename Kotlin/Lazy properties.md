# **Lazy properties**

```kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main() {
    println(lazyValue) // 첫 실행. 람다 실행 후 결과를 기억.
    println(lazyValue) // 후속호출(get()) 기억된 결과를 반환.
}

실행결과
computed!
Hello
Hello
```

- `lazy`는 람다를 사용하여 Lazy<T> 인스턴스를 반환하는 함수임.
    - 이는 지연 초기화를 위한 대리 역할임.
- 첫 호출로 `lazy`에 전달된 람다를 실행하고 결과를 기억함.
- `get()` 에 대한 후속 호출은 단순히 기억된 결과를 반환함.
    - 여기서 후속 호출이란 첫 실행 이후를 말하는 것임.
        - `get()` 이라 표현한 코틀린에서 프로퍼티 사용시 read는 해당 프로퍼티를 그냥 사용하는 것임.

### lazy의 파라미터 - LazyThreadSafetyMode.PUBLICATION

```kotlin
public actual fun <T> lazy(mode: LazyThreadSafetyMode, initializer: () -> T): Lazy<T> =
    when (mode) {
        LazyThreadSafetyMode.SYNCHRONIZED -> SynchronizedLazyImpl(initializer)
        LazyThreadSafetyMode.PUBLICATION -> SafePublicationLazyImpl(initializer)
        LazyThreadSafetyMode.NONE -> UnsafeLazyImpl(initializer)
    }

public actual fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)
```

- 기본적으로 지연 초기화는 동기화 됨. → default는 `LazyThreadSafetyMode.SYNCHRONIZED`임.
    - 값은 하나의 스레드에서만 계산되지만 모든 스레드에는 동일한 값이 표시됨.
- 여러 스레드에서 동기화가 필요하지 않은 경우 `LazyThreadSafetyMode.PUBLICATION` 을 `lazy`의 파라미터에 전달.
    - 다른 스레드에서 새롭게 동기화 되는 걸 알 수 있음.

```kotlin
val lazyValue: String by lazy(LazyThreadSafetyMode.PUBLICATION) {
    println("computed!")
    "Hello"
}

fun main() = runBlocking<Unit> {
    GlobalScope.launch {
        println(lazyValue)
    }
    GlobalScope.launch {
        println(lazyValue)
    }
}

실행결과
computed!
Hello
computed!
Hello
```

### lazy의 파라미터 - LazyThreadSafetyMode.NONE

- 초기화가 항상 프로퍼티를 사용하는 스레드와 동일한 스레드에 이루어 질 것이라고 확신하는 경우 사용.
- `LazyThreadSafetyMode.NONE` 를 사용하는 경우 스레드 안전 보장 및 오버헤드가 발생하지 않음.

```kotlin
abstract class BaseActivity<T : ViewDataBinding>(
    @LayoutRes private val layoutId: Int
    ) : AppCompatActivity() {
    protected val binding: T by lazy(LazyThreadSafetyMode.NONE) {
        DataBindingUtil.setContentView(this, layoutId)
    }

    init {
        addOnContextAvailableListener {
            binding.notifyChange()
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding.lifecycleOwner = this
    }

    override fun onDestroy() {
        binding.unbind()
        super.onDestroy()
    }

    protected inline fun bind(block: T.() -> Unit) {
        binding.apply(block)
    }
}
```

- 위 코드의 binding 프로퍼티는 항상 main 스레드에서 초기화 되고, main 스레드에서만 사용됨.
