# Hilt로 생성되는 코드 확인하기

![image](https://github.com/jiwon2724/TIL/assets/70135188/e9eb477a-688e-478f-856d-a85c005411f7)


- 어노테이션을 어떻게 사용하느냐에 따라서 생성되는 코드가 달라지게 되므로, 이를 확인해보자.

## @HiltAndroidApp

- 힐트의 출발점은 언제나 `@HiltAndroidApp` 어노테이션임.
- `@HiltAndroidApp` 를 Application Class에 마킹하고, 빌드를 수행하면 기본적인 설정은 끝임.
    - 빌드 이후엔 `SingletonComponent` 가 자동으로 생성됨.
        - 이는 DI 컨테이너임.
        - 여러가지 의존성을 추가하는 기법을 활용하여 컴파일 타임에 `SingletonComponent` 에 의존성을 추가할 수 있음.
            - 의존성 추가기법은 다음과 같음.
                - 생성자 바인딩
                    - 바인딩이란 컴포넌트에 의존성을 추가하는 것. or 바인딩을 한다
                - `@Provides` 어노테이션 마킹
                - 등

## @AndroidEntryPoint

- `@AndroidEntryPoint` 는 반드시 `@HiltAndroidApp` 선언 이후에 Activity, Fragment, Service등 컴포넌트들에 마킹할 수 있음.
- `@AndroidEntryPoint` 를 Activity에 마킹했다면, `ActivityRetainedComponent` 가 생성됨.
    - 수동 의존성 주입에서 AppContainer와 LoginContainer를 각각 만들어서 사용했지만, Hilt를 사용하므로  `@HiltAndroidApp` 과 `@AndroidEntryPoint` 두 어노테이션으로 컨테이너를 분리하여 생성함.

## @Inject

- 의존성을 주입할 땐 `@Inject` 어노테이션을 사용함.
    - 즉, 의존성 주입을 요청함.
- 생성자에 `@Inject` 어노테이션을 붙이는 경우는 해당 클래스가 사용되는 컴포넌트에 바인딩 한다는 의미임.
    - 즉, 의존성을 추가 하겠다는 뜻
- 어노테이션의 이름에 따라서 주입하는 컴포넌트가 달라짐.
    - ex) `@HiltAndroidApp` 이 마킹된 클래스에서 사용했다면, `SingletonComponent` 에 주입됨.
    - ex) `@AndroidEntryPoint` 가 마킹된 클래스에서 사용했다면, `ActivityRetainedComponent` 에 주입됨.

```kotlin
@HiltAndroidApp
class App : Application() {
    @Inject
    lateinit var car: Car

    override fun onCreate() {
        super.onCreate()

        Log.d("Car Name : ", "$car")
    }
}
```

```kotlin
class Car @Inject constructor() {
    override fun toString(): String {
        return "Casper"
    }
}
```

## @Module

- `@Module` 어노테이션을 통해 바인딩을 할 수 있음.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    fun provideCarName(): Car {
        return Car()
    }
}
```

- `@InstallIn` 을 통해서 `SingletonComponent` 에 AppModule object가 설치되는걸 마킹함.
- `@Module`을 통해 바인딩 해준다면, 생성자 바인딩은 안해도 무관함.
    - `@Module` 에서 제공하는 `@Provides` provideCarName()가 제공됨.

```kotlin
class Car @Inject constructor() {
    override fun toString(): String {
        return "Casper"
    }
}
```

## 어노테이션으로 인해 생성되는 코드

- 위 코드를 통해서 생성되는 Hilt 코드는 다음과 같음.
  - 더 많지만 생략.

```kotlin
@QualifierMetadata
@DaggerGenerated
@Generated(
    value = "dagger.internal.codegen.ComponentProcessor",
    comments = "https://dagger.dev"
)
@SuppressWarnings({
    "unchecked",
    "rawtypes"
})
public final class App_MembersInjector implements MembersInjector<App> {
  private final Provider<Car> carProvider;

  public App_MembersInjector(Provider<Car> carProvider) {
    this.carProvider = carProvider;
  }

  public static MembersInjector<App> create(Provider<Car> carProvider) {
    return new App_MembersInjector(carProvider);
  }

  @Override
  public void injectMembers(App instance) {
    injectCar(instance, carProvider.get());
  }

  @InjectedFieldSignature("toss.next.hiltpractice.App.car")
  public static void injectCar(App instance, Car car) {
    instance.car = car;
  }
}
```

```kotlin
  @Override
  public void injectMembers(App instance) {
    injectCar(instance, carProvider.get());
  }

  @InjectedFieldSignature("toss.next.hiltpractice.App.car")
  public static void injectCar(App instance, Car car) {
    instance.car = car;
  }
```

- 위 코드를 보면  App에 car를 주입하고 있음.

```kotlin
@Inject
lateinit var car: Car
```

- `injectCar` 는 결국 Application에 있는 car에 값을 할당하게함.

```kotlin
  @Override
  public void injectMembers(App instance) {
    injectCar(instance, carProvider.get());
  }
```

- 주어진 프로바이더에서 get함수를 통하여 얻게됨.
