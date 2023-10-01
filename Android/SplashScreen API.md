# SplashScreen

- Android 11 이하에서 스플래쉬 액티비티를 구현했다면, 12이상에선 스플래쉬가 2번 보여진다.
- Android 12부턴 시스템은 항상 모든 앱 시작시 Android 시스템 기본값 스플래시 화면을 적용한다.

### 스플래시 화면 작동 방식

사용자가 앱을 실행할 때 앱 프로세스가 실행되지 않거나(시스템에 앱 프로세스가 없는 경우 `콜드 스타트`), 액티비티가 만들어지지 않은 상태라면(`웜 스타트`) 다음 이벤트가 발생한다. 

1. 시스템은 개발자가 정의한 테마와 애니메이션을 사용하여 스플래시 화면을 표시한다.
2. 앱이 준비되면 스플래시 화면이 닫히고 앱이 표시된다. 

```kotlin
💡 스플래시 화면은 핫 스타트 중에 표시되지 않는다.
```

### SplashScreen 라이브러리

```kotlin
build.gradle

android {
   compileSdkVersion 31
   ...
}
dependencies {
   ...
   implementation 'androidx.core:core-splashscreen:1.0.0-alpha01'
}
```

### SplashScreen 적용

```kotlin
<style name="Theme.Splash" parent="Theme.SplashScreen">
    <item name="windowSplashScreenBackground">@color/white</item>
    <item name="windowSplashScreenAnimatedIcon">@drawable/git_logo</item>
    <item name="postSplashScreenTheme">@style/Theme.Clean_rchitecture_mvi_practice</item> 
</style>
```

- `windowSplashScreenBackground` : 특정 단색으로 배경을 채운다.
- `windowSplashScreenAnimatedIcon` : 지정한 drawable로 시작 창 중앙의 아이콘을 대채한다.
    - `AnimationDrawable` 과 `AnimatedVectorDrawble` 을 통해 애니메이션이 가능하고, 애니메이션이 재생되도록 `windowSplashScreenAnimationDuration` 을 설정할 수 있다.
- `windowSplashScreenBrandingImage` : 스플래시 화면 하단에 표시할 이미지를 설정할 수 있다.
- `postSplashScreenTheme` : 스플래시가 끝난 후 보일 화면의 테마이다.

위 속성 말고 다양한 속성들이 있는데, 관련내용은 공식문서를 참고해주세요!

1. 매니페스트 `application` 태그에서 만들어놓은 테마 속성을 지정.

```kotlin
<application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Splash">
				...
</application
```

2. 스플래쉬를 보여줄 Activity의 `setContextView` 를 호출하기 전에 `installSplashScreen()` 을 호출.

```kotlin
class MainActivity : AppCompatActivity() {
    lateinit var binding: ActivityMainBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        installSplashScreen()
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
    }
}
```

```kotlin
💡 `installSplashScreen()` 은 스플래시 화면 객체를 반환하며 이 객체를 사용하여
    애니메이션을 맞춤설정하거나 화면에 스플래시 화면을 더 오래 표시할 수 있음.
```
