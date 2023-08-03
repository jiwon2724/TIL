# Hilt

- 의존성 주입의 상용구 코드를 줄여주는 안드로이드용 DI 라이브러리
- DI Container를 제공하고 생명주기를 자동으로 관리하며 DI를 사용하는 방법을 제시

```kotlin
Hilt는 Dagger2 기반의 라이브러리며 표준화된 Dagger2 사용법을 제시하고 보일러 플레이트 코드를 감소시킴
```

# Koin vs Dagger2 vs Hilt

다른 DI 라이브러리가 있지만 왜 Hilt를 사용할까?

1. Hilt는 Android Jetpack 권장 라이브러리임.
2. Dagger2보다 러닝커브는 적고, Dagger2가 제공하는 이점을 가져옴
    1. Hilt는 Dagger2기반 라이브러리임.
        1. 컴파일 타임 정확성
        2. 런타임 성능
3. Koin은 런타임 시 주입인 방면 Hilt는 컴파일 타임에 주입에 대한 검증을 마침.
    1. 에러 검출 시점이 컴파일 타임이라 안전하게 사용할 수 있음.

위 같은 이유로 Hilt는 안드로이드 DI 라이브러리로 많이 사용함.

# @HiltAndroidApp

- Hilt를 사용하는 모든 앱은 Application Class에 `@HiltAndroidApp` 어노테이션을 지정해야함.

```kotlin
@HiltAndroidApp
class MemoApplication: Application() {
    override fun onCreate() { .. }
}
```

- 이는 모든 의존성 주입의 시작점이며, Hilt 코드를 생성하고 SingletonComponent를 생성 및 인스턴스 화함.
- 컴포넌트(SingletonComponent)를 인스턴스화 하는 부분은 onCreate 메서드에서 수행함.
    - 이는 바이트 코드로 변환되며, 컴파일 시 Hilt접두어가 붙은 Application Class를 생성함
    
    ```kotlin
    class MemoApplication : Hilt_MemoApplication
    ```
    
    - 생성된 Hilt 클래스는 컴포넌트 인스턴스화 및 DI관련 코드를 포함함.
    - Hilt_XXX 클래스를 상속해야할 것 같지만, 컴파일 시 바이트 코드를 산출하고 그 이후에 Gradle 플러그인이 개입해 바이트 코드를 조작하여 XXXAplication은 Hilt_XXXAplication을 상속하는 코드로 변환됨.
        - 이는 개발자의 편의성을 위하여 도모됨.
        - 바이트 코드 변환을 원하지 않는다면 Gradle 플러그인을 비활성화.
            - 이렇게 됐을경우 Hilt_XXApplication을 하드코딩하여 상속시켜야함.

# @InstallIn

- Hilt가 생성하는 DI 컨테이너에 어떤 모듈을 사용할지 명시해야함.
- 해당 어노테이션에 들어오는 컴포넌트에 모듈이 설치됨.
- Hilt는 이를보고 컴파일타임에 관련 코드를 생성함.
- 올바르지 않은 스코프와 컴포넌트를 설정시 에러를 발생시킴.

```kotlin
class MemoRepository @Inject constructor(
    private val db: MemoDatabase
) {
    ...
}
```

```kotlin
@InstallIn(SingletonComponent::class)
@Module
object DataModule {
    @Provides
    fun provideMemoDB(@ApplicationContext context: Context) = 
        Room.databaseBuilder(context, MemoDatabase::class.java, "Memo.db").build()
}
```

- 만약 ActivityComponent와 FragmentComponent 둘 다 필요한경우엔 상위 컴포넌트에 설치하는걸 고려해야함.
- `SingletonComponent` 에 DataModule이 생성됨.

# @AndroidEntryPoint

- 해당 어노테이션이 추가된 클래스에 DI Container를 추가함.
    - HiltAndroidApp 설정 후 사용 가능함.
- Activity, Fragment, Service, View, BroadcastReceiver에만 지원함.
    - 사용시 알맞는 컴포넌트가 생성됨.
- 액티비티에 선언된 `@Inject` 어노테이션이 달린 변수에 대해 DI를 수행함
- SingletonComponent의 하위 컴포넌트인 ActivityComponent가 생성됨.
    - 이는 `SingletonComponent` 에 생성된 DI Container의 항목을 주입받을 수 있음.

```kotlin
@AndroidEntryPoint
class MemoActivity : AppCompatActivity() {
    @Inject lateinit var repository: MemoRepository
}
```

위 코드들의 객체 그래프는 다음과 같음.

![image](https://github.com/jiwon2724/TIL/assets/70135188/4cebcf36-16e2-45c5-8208-3f6797c0c0b3)


- 컴파일 타임에 주입의 검증을 마치게됨.

# Hilt Component Hierarchy

![image](https://github.com/jiwon2724/TIL/assets/70135188/355e13fd-50b0-4497-ad23-0180ec4f5363)


- 힐트는 이미 정의된 표준화된 컴포넌트 세트를 제공함.
    - 이는 각 생명주기와 기능에 알맞는 컴포넌트와 스코프임.
- `@AndroidEntryPoint`를 사용하여 해당 타입의 알맞는 컴포넌트를 추가하게됨.
- 화살표 방향 기준으로 상위에서 하위로 가는 컴포넌트를 의미함.
    - 하위 컴포넌트는 상위 컴포넌트의 의존성에 접근할 수 있음.
        - 직계 관계에서만 접근 가능
- 모듈 클래스에 스코프 어노테이션을 사용하여 동일 인스턴스를 공유할 수 있음.

```kotlin
@Singleton
class MemoRepository @Inject constructor(
    private val db: MemoDatabase
) {
    ...
}
```

- 위 처럼 MemoRepository 클래스를 `@Singleton` 으로 설정하게 되면, 동일한 인스턴스를 주입받을 수 있음.
- 모듈에서 사용하는 스코프 어노테이션은 반드시`@InstallIn`에 명시된 컴포넌트와 쌍을 이루는 스코프를 사용해야함.
- Hilt는 자원 공유를 쉽게 하도록 도와줌.
