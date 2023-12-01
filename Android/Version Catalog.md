# Version Catalog

- 라이브러리 이름과 버전 정보를 지정하여 빌드 시스템 전반에 활용할 수 있는 기능
- Gradle 7.0 이상부터 사용 가능하고, 7.4부터 안정화됨.
- 버전 카탈로그 파일을 만들고, 다음과 같은 섹션을 추가해야함.

```kotlin
// libs.versions.toml

[versions]
kotlinx-coroutines = "1.7.1"
kotlin = "1.8.21"

[libraries]
kotlinx-coroutines-android = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-android", version.ref = "kotlinx-coroutines" }

[plugins]
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
```

- 위 처럼 버전 정보와 라이브러리, 플러그인 이름을 지정할 수 있음.

```kotlin
// app/build.gradle.kts

dependencies {
    implementation(libs.kotlinx.coroutines.android)
}
```

- `build.gradle.kts`에서 `libs.versions.toml` 에 지정한 정보를 이용해 의존성을 지정할 수 있음.

```kotlin
versionCatalogs {
    create("libs") {
        from(files("../gradle/libs.versions.toml"))
    }
}
```

- 원래라면 위와 같이 버전 카탈로그를 이용하겠다는 dsl문을 작성해 줘야함.
- 전역에서 libs를 사용할 수 있어서 위 dsl은 작성을 안해줘도됨.
- 즉, libs는 버전 카탈로그 인스턴스임.

# BuildSrc 말고 왜 Version Catalog?

- buildSrc의 상수값을 수정 한다면, 빌드 스크립트의 재컴파일이 필요함.
- buildSrc를 사용하면서, 의존성 버전정보를 번경하면 빌드 오버헤드가 발생할 수 있음.
    - 이는 빌드 시간이 느려지는 문제로 직결됨.
- buildSrc로도 충분히 버전 정보를 표현 가능하지만, 버전 카탈로그를 사용하는 것이 의존성 관리에 편안하고, 빌드 성능도 올려줄 수 있음.
