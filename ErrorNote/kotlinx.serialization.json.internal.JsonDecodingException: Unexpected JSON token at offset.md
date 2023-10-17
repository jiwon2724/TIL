# kotlinx.serialization.json.internal.JsonDecodingException: Unexpected JSON token at offset
# 문제상황
```kotlin
FATAL EXCEPTION: main
Process: com.poketmonencyclopedia, PID: 2854
kotlinx.serialization.json.internal.JsonDecodingException: Unexpected JSON token at offset 334: Encountered an unknown key 'evolves_from_species' at path: $.evolution_chain
Use 'ignoreUnknownKeys = true' in 'Json {}' builder to ignore unknown keys.
JSON input: ...../api/v2/evolution-chain/1/"},"evolves_from_species":null,"fl.....
    at kotlinx.serialization.json.internal.JsonExceptionsKt.JsonDecodingException(JsonExceptions.kt:24)
    at kotlinx.serialization.json.internal.JsonExceptionsKt.JsonDecodingException(JsonExceptions.kt:32)
    at kotlinx.serialization.json.internal.AbstractJsonLexer.fail(AbstractJsonLexer.kt:598)
    at kotlinx.serialization.json.internal.AbstractJsonLexer.failOnUnknownKey(AbstractJsonLexer.kt:593)
    at kotlinx.serialization.json.internal.StreamingJsonDecoder.handleUnknown(StreamingJsonDecoder.kt:254)
    at kotlinx.serialization.json.internal.StreamingJsonDecoder.decodeObjectIndex(StreamingJsonDecoder.kt:240)
    at kotlinx.serialization.json.internal.StreamingJsonDecoder.decodeElementIndex(StreamingJsonDecoder.kt:175)
    at com.poketmonencyclopedia.detail.model.species.SpeciesResponse$$serializer.deserialize(SpeciesResponse.kt:5)
    at com.poketmonencyclopedia.detail.model.species.SpeciesResponse$$serializer.deserialize(SpeciesResponse.kt:5)
...
```
request <-> response에서 kotlin-serialize 라이브러리 사용 중 발생함.

# 원인
서버의 response를 받는 데이터 포맷(json)이 일치하지 않음.

# 참고사항
 - response로 주는 데이터가 매우 컸고, 타입도 파라미터 값에 달라졌다. 구현하는 기능에 굳이 해당 데이터가 필요하지 않았음.
 - 해당 key를 꼭 포함해야 하는지 의문이 생겼고, 직렬화/역직렬화 라이브러리(kotlin-serialize)의 기본 동작은 모든 필드를 기대하고 있기 때문에 해당 필드가 누락되면 에러를 발생시키는 걸 확인함.

# ❌ 첫 번째 시도)
```kotlin
@Serializable
data class SpeciesResponse(
    val base_happiness: Int,
    val capture_rate: Int,
    val color: Color,
    val egg_groups: List<EggGroup>,
    val evolution_chain: EvolutionChain,
    val evolves_from_species: Any,
    val flavor_text_entries: List<FlavorTextEntry>,
    val form_descriptions: List<Any>,
    val forms_switchable: Boolean,
    val gender_rate: Int
    ...
)
```
- 타입이 request 파라미터마다 달라졌고, 기능 구현에 사용하지 않는 데이터기에 Any 타입으로 처리하려고 시도했다.
- 하지만 모든 Kotlin 객체는 Any의 서브타입 이기에 따라서 Any 타입의 프로퍼티가 있을 경우 직렬화 시스템은 그 프로퍼티가 어떤 실제 타입인지 알 수 없었다.

# ❌ 두 번째 시도)
```kotlin
data class SpeciesResponse(
    val base_happiness: Int,
    val capture_rate: Int,
    val color: Color,
    val egg_groups: List<EggGroup>,
    val evolution_chain: EvolutionChain,
    @Contextual val evolves_from_species: Any,
    val flavor_text_entries: List<FlavorTextEntry>,
    val form_descriptions: List<@Contextual Any>,
    val forms_switchable: Boolean,
    val gender_rate: Int
    ...
)
```
- `@Contextual` 어노테이션 사용.
- 위 어노테이션은 kotlinx.serialization 라이브러리에서 사용되는데, `@Contextual` 어노테이션을 추가하면, 직렬화 및 역직렬화의 동작이 Serializer 컨텍스트에 따라 결정됨.
  - Serializer 필요한 직렬화 동작이나 설정을 관리하는 데 사용됨.
- 에러(빨간줄)는 사라졌지만, 빌드 시 같은 에러 발생.
  - Any는 구체적인 타입 정보를 가지지 않기 때문에 직렬화 및 역직렬화를 처리할 때 문제가 생김.
  - `@Contextual` 어노테이션은 특정 타입에 대해 커스텀 직렬화 동작을 제공하려는 경우에 사용됨. 그런데 Any 타입의 경우 어떤 실제 타입이 할당될지 알 수가 없음.
# ✅ 세 번째 시도)

- 에러 내용중 Use 'ignoreUnknownKeys = true' in 'Json {}' builder to ignore unknown keys. 키워드를 발견.
- `'ignoreUnknownKeys = true'`는 JSON 문자열에 알 수 없는 키가 포함되어 있을 경우 무시하도록 하는 옵션임.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Singleton
    @Provides
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        val jsonConfig = Json { ignoreUnknownKeys = true } // 기존엔 없었음.
        return Retrofit.Builder()
            .client(okHttpClient)
            .baseUrl("https://pokeapi.co/api/v2/")
            .addConverterFactory(jsonConfig.asConverterFactory("application/json".toMediaType()))
            .build()
    }
```

# 회고 및 정리

**kotlin-serialization 직렬화 라이브러리 사용시엔 json 데이터가 일치해야 하는 것을 확인할 것.**
1. 공공 api, 무료로 배포되어 있는 api의 문서를 잘 보고 타입 맞추기.
2. 사용 안하는 데이터일 경우 `ignoreUnknownKeys = true` 추가.
3. kotlinx.serialization에서 지원하지 않는 직렬화, 역직렬화 타입이라면 `@Contextual`사용하기.
   - 대부분 `@Serializable` + data class로 사용 가능함.
