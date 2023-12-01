# Convention Plugin

- 안드로이드 멀티 모듈 프로젝트에서는 규모에 따라 모듈이 점점 증가한다.
- 모듈이 늘어날 때 마다 build.gradle 스크립트 파일들도 점점 증가한다.
- build.gradle 스크립트 파일과 같이 재사용이 가능한 중복 코드들이 많이 있음.
- 위와 같은 스크립트의 중복 코드들을 하나의 빌드 로직으로 재사용할 수 있음.

# Gradle Plugin

- 빌드 로직을 재사용할 수 있는 확장 기능임.
- Gradle Task를 추가하거나, 빌드 로직 코드들을 별도로 관리함.
- Script Plugin, Binary Plugin, Precompiled Script Plugin 세 가지 방법이 있음.

### Script Plugin

- 스크립트 코드를 별도 파일로 분리

```kotlin
apply(from = "common.gradle") // 중복된 코드가 담긴 파일
apply(from = "project.gradle") // 중복된 코드가 담긴 파일
```

- 중복되는 스크립트 코드들을 복사해서 Groovy 파일로 다시 만들어야함.
    - Groovy로만 작성해야하는 이유는 `gradle.kts`를 사용하는 코틀린 스크립트는 `build.gradle`이라는 파일 이름을 인식하여 코드를 빌드함.
    - XXX.gradle 파일과 같은 스크립트 파일과 같은 형태는 인식하지 못함.
        - 위 코드에선 common.gradle, project.gradle
- `apply`를 사용해서 파일을 적용.

### Binary Plugin

- Plugin 인터페이스를 구현하는 클래스 작성
- 플러그인 아이디를 등록해서 사용

```kotlin
plugins {
    id("my.custom.plugin")
}
```

- 의존성에 있는 라이브러리들의 플러그인들을 작성해야함.
    - ex) hilt, kotlin, android
- 의존성 뿐만 아니라, 안드로이드 버전 관련 설정코드도 작성할 수 있음.

```kotlin
internal class AndroidLibraryPlugin : Plugin<Project> {
    override fun apply(target: Project) = with(target) {
        with(pluginManager) {
            apply("com.androidLibrary")
            apply("com.jetbrains.kotlin.android")
        }
        extensions.configure<LibraryExtension> {
            compileSdk = 33
            minSdk = 24
            compileOptions {
                sourceCompatibility JavaVersion.VERSION_1_8 
                targetCompatibility JavaVersion.VERSION_1_8
            }
            kotlinOptions {
                jvmTarget = '1.8'
            }
        }
    depedencies { ... }
    }
}
```

- `extensions` 은 build.gradle에 있는 `android { .. }` 블록에 접근하기 위해 사용된 코드임.
- 버전 카탈로그를 사용한다면 다음과 같이 libs를 참조하여 가져와야함.

```kotlin
val libs = extensions.getByType<VersionCatalogExtension>().named("libs")
depedencies {
    add("coreLibraryDesugaring", libs.findLibrary("android.desugarJdkLibs").get())
}
```

- 모듈을 만들게되면 중복되는 코드가 많이 생성될텐데 위 코드처럼 중복되는 부분을 적용한다면 다음과 같이 한 줄이면 됨.

```kotlin
plugins {
    id("convention.android.feature")
}
```

### Precompiled Script Plugin

- Binary Plugin을 클래스로 작성하는 대신  gradle.kts 파일의 스크립트 형태로 간편하게 작성할 수 있는 확장 플러그인
- Script Plugin과 Binary Plugin의 특징을 모두 사용함.

```kotlin
// build.gradle.kts
plugins {
    id("common.android")
}
```

- 적용해보구 업데이트 하기.
