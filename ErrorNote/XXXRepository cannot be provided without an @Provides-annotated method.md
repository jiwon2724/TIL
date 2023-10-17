# XXXRepository cannot be provided without an @Provides-annotated method.

# 문제상황
```kotlin
/Users/jiwon_dev/AndroidStudioProjects/PoketmonEncyclopedia/app/build/generated/hilt/component_sources/debug/com/poketmonencyclopedia/PoketmonEncyclopedia_HiltComponents.java:136: error: [Dagger/MissingBinding] com.poketmonencyclopedia.domain.detail.PoketmonDetailRepository cannot be provided without an @Provides-annotated method.
  public abstract static class SingletonC implements PoketmonEncyclopedia_GeneratedInjector,
                         ^
  
  Missing binding usage:
      com.poketmonencyclopedia.domain.detail.PoketmonDetailRepository is injected at
          com.poketmonencyclopedia.domain.detail.PoketmonDetailUseCase(poketmonDetailRepository)
      com.poketmonencyclopedia.domain.detail.PoketmonDetailUseCase is injected at
          com.poketmonencyclopedia.poketmondetail.PoketmonDetailViewModel(poketmonDetailUseCase)
      com.poketmonencyclopedia.poketmondetail.PoketmonDetailViewModel is injected at
          com.poketmonencyclopedia.poketmondetail.PoketmonDetailViewModel_HiltModules.BindsModule.binds(arg0)
      @dagger.hilt.android.internal.lifecycle.HiltViewModelMap java.util.Map<java.lang.String,javax.inject.Provider<androidx.lifecycle.ViewModel>> is requested at
          dagger.hilt.android.internal.lifecycle.HiltViewModelFactory.ViewModelFactoriesEntryPoint.getHiltViewModelMap() [com.poketmonencyclopedia.PoketmonEncyclopedia_HiltComponents.SingletonC → com.poketmonencyclopedia.PoketmonEncyclopedia_HiltComponents.ActivityRetainedC → com.poketmonencyclopedia.PoketmonEncyclopedia_HiltComponents.ViewModelC]
```

토이 프로젝트에 모듈화 + 클린 아키텍처를 적용하기 위해 hilt module을 적용하다 위 에러를 직면했다.

# 원인

data 모듈에서 hilt plugin과 dependency를 추가를 안해준 것.

# ❌ 첫 번째 시도)

에러내용 자체가 hilt module관련된 내용이라 hilt module을 꼼꼼히 살펴봤지만, 아무 이상이 없었다.

# ❌ 두 번째 시도)

구글링 해보니 app 모듈에 data 모듈을 추가를 안해줘서 추가를 해줬지만, 변함이 없었다.

# ✅ 세 번째 시도)

data 모듈의 gradle을 꼼꼼히 살펴보니 hilt 관련 plugin과 dependency가 누락된 걸 확인했다.

```kotlin
plugins { 
    kotlin("plugin.serialization") version "1.5.0"
}

dependencies {
    implementation(libs.hilt.android)
    kapt(libs.hilt.compiler)
}

// 위 코드를 추가 안해줬음.
```

# 회고 및 정리

**cannot be provided without an @Provides-annotated method. 등 해당 단어들이 포함될 경우 아래사항을 체크해보자.**

1. app 모듈에서 잘 참조하고 있는지 확인.
2. hilt module에 `@Provides` 어노테이션 누락 확인.
3. data module에 hilt 관련 plugin, dependency 확인.
