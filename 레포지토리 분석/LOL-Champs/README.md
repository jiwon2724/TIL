https://github.com/KimReady/LOL-Champs

```
app/src/main/java
└── com
    └── ready
        └── lolchamps
            ├── LolChampsApplication.kt
            ├── db
            │   ├── AppDatabase.kt
            │   ├── ChampionDao.kt
            │   ├── ChampionInfoDao.kt
            │   └── TypeConverters.kt
            ├── di
            │   ├── DBModule.kt
            │   ├── NetworkModule.kt
            │   └── RepositoryModule.kt
            ├── exceptions
            │   └── Exceptions.kt
            ├── model
            │   ├── Champion.kt
            │   ├── ChampionInfo.kt
            │   ├── ChampionResponse.kt
            │   └── Image.kt
            ├── network
            │   ├── ChampionInfoService.kt
            │   ├── ChampionService.kt
            │   └── RequestDebugInterceptor.kt
            ├── repository
            │   ├── DetailRepository.kt
            │   ├── DetailRepositoryImpl.kt
            │   ├── MainRepository.kt
            │   └── MainRepositoryImpl.kt
            ├── ui
            │   ├── base
            │   │   ├── BaseActivity.kt
            │   │   └── UiState.kt
            │   ├── detail
            │   │   ├── DetailActivity.kt
            │   │   ├── DetailViewModel.kt
            │   │   └── SkinAdapter.kt
            │   └── main
            │       ├── ChampionAdapter.kt
            │       ├── ChampionItemDecoration.kt
            │       ├── MainActivity.kt
            │       └── MainViewModel.kt
            └── utils
                ├── BindingAdapters.kt
                ├── DimensionUtils.kt
                └── UriHelper.kt
```

- `db` : Room 라이브러리 관련 패키지. DB 생성을 위한 AppDatabase, DAO 클래스 등이 있음.
- `di` : local, remote에 필요한 hilt module이 있음.
- `exceptions` : api 통신 실패시 예외처리 래핑 클래스.
- `model` : local, remote에 사용되는 데이터 타입.
- `network` : REST API 통신을 위한 인터페이스 등이 있음.
- `repository` : local, remote 통신을 위한 구현체.
- `ui` : 메인, 디테일 화면 및 어댑터 등
- `utils` : 구현에 필요한 확장함수들임.

