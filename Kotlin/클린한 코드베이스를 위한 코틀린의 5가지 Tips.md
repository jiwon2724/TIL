# 클린한 코드베이스를 위한 코틀린의 5가지 Tips
해당 포스팅은 https://medium.com/@domen.lanisnik/5-kotlin-tips-for-a-cleaner-codebase-3582f2e4e2af 번역한 글입니다.

- 코틀린은 간결한 코드를 쉽게 작성할 수 있는 개념과 구조를 많이 제공한다.
- 그러나 팀 단위로 작업 시 주요 목표는 읽고, 이해하고, 유지보수에 쉬운 코드를 작성하는 것이다.
- 보다 좋은 코드베이스를 유지하는 몇가지 모범 사례를 살펴보자.


**💡 이는 단지 제안일 뿐이며, 올바른 방법임을 의미하지는 않음. 코드 스타일은 팀의 선호도에 따라 다름.**

### 1. Class의 가시성에 주의하자
- 새로운 클래스와 함수에 적용하는 접근 제어자에 주의를 기울일 것.
- 클래스는 기본적으로 public임. 이는 해당 클래스에 의존하는 다른 모듈에서 클래스를 엑세스할 수 있음.

**접근 제어자**
- `public` : 모듈 내부의 모든 클래스와 이 모듈에 의존하는 모든 모듈에 표시되는 기본 수정자
  - default임.
- `internal` : 모듈 내부의 모든 클래스에 표시되지만 모듈 외부에는 표시되지 않음.
- `private` : 파일이나 클래스 내부에서만 볼 수 있음.
- `protected` : 클래스의 멤버(함수, 프로퍼티)도 이를 상속하는 모든 클래스에 표시됨.
  - `oepn`사용.
- 클래스가 현재 모듈 내부로만 표시되도록 제한하려면 가능할 때 마다 `internal` 접근 제어자를 사용할 것.
  - 이는 모듈의 외부 API가 줄어듬.

### 2. 최상위 선언을 최소한으로 유지하자.
- 최상위 함수(클래스 외부에 존재하는 함수)는 클래스를 선언할 필요 없이 Helper, Utility 함수를 정의하는데 매우 유용함.
- 확장함수는 우리가 소유하지 않은 클래스의 기능을 상속하거나, 디자인 패턴을 사용하지 않고 확장할 수 있게 해줌.
- 확장 기능의 범위, 가시성을 상황에 맞는 파일, 클래스 또는 모듈로 제한할 것.

```kotlin
// top-level declaration
fun String.isValidUsername(): Boolean {
    return this.matches(Regex("^[a-zA-Z0-9._-]{3,15}\$"))
}
```
- 위 확장함수는 공개적이고 모듈 어디서나 액세스할 수 있음.
- 문자열에서 함수를 호출하고 싶을 때마다 관련 없는 `isValidUsername` 함수가 제안 목록에 표시됨.
- 위 같은 유형의 함수가 많이 있으면 개발자 환경이 저하되고, 제안 내용과 관련이 없어짐.

### 3. 몇 줄의 코드를 작성하는 것보다 가독성을 선호하자.
- Kotlin은 한 줄로 여러 작업을 쉽게 수행할 수 있는 강력한 기능을 제공함. 그러나 이는 다른 개발자가 코드를 읽기가 더 어렵게 만듦.
- 몇 줄의 추가 코드가 필요하더라도 복잡한 연결 연산자보다 명확하고 간단한 구문을 선호.

```kotlin
함수 체이닝
private fun squareIfPositive(someNumber: Int): Int {
    return someNumber.takeIf { it > 0 }?.let { it * it } ?: 0
}

// or

private fun squareIfPositive(someNumber: Int): Int {
    return someNumber
        .takeIf { it > 0 }
        ?.let { it * it }
        ?: 0
}
```

```kotlin
자바와 유사한 방식
// option 1
private fun squareIfPositive(someNumber: Int): Int {
    return if (someNumber > 0) {
        someNumber * someNumber
    } else {
        0
    }
}

// option 2
private fun squareIfPositive(someNumber: Int): Int {
    if (someNumber <= 0) {
        return 0
    }
    return someNumber * someNumber
}

```
- 한 줄로 작성한 만큼 좋진 않지만, 자바와 유사한 방식의 코드는 누구든지 읽고 이해하기가 더 쉬움.
- 소프트웨어 엔지니어로서 우리는 코드를 작성하는 것보다 코드를 읽는데 더 많은 시간을 소비하므로 팀원과 미래의 자신을 위해 더 쉽게 만들 수 있음.

### 4. Pair, Triple을 사용하는 것보다 data class생성을 선호하자.
- 둘 또는 세 개의 값을 반환해야 할 때 Pair, Triple 클래스는 도움이 될 수 있음.
- 그러나 반환되는 값이 어떤 것을 의미하는지 다른 개발자들은 이해하지 못함.
```kotlin
suspend fun authenticateUser(username: String): Pair<String, String> {
    // perform authentication logic
    return Pair("accessToken", "refreshToken")
}
```
- 위 코드에서 `authenticateUser` 함수를 사용할 경우 `authenticateUser.first()`와 같이 사용해야함.

```kotlin
suspend fun authenticateUser(username: String): AuthenticationTokens {
    // perform authentication logic
    return AuthenticationTokens(
        accessToken = "accessToken",
        refreshToken = "refreshToken"
    )
}

data class AuthenticationTokens(
    val accessToken: String,
    val refreshToken: String
)
```
- 위와 같이 사용하면 명시적으로 반환되는 내용과 각 값이 나타내는 내용이 더 명확해짐.

### 5. 철저한 when(state, expression)을 선호하자.
- 제한된 클래스 계층 구조의 값을 확인하기 위해 분기를 포괄적으로 사용하는 대신 가능한 모든 값을 저의하는 것이 좋음.
  - `enum class`, `sealed class`, `sealed interface`
- `else`는 새 값이 추가될 때 잠재적인 버그가 발생할 수 있음.

```kotlin

enum class ProfileAnalyticsEvent {
    PROFILE_OPENED,
    PROFILE_EDITED,
    PROFILE_SAVED,
    PROFILE_DELETED
}

fun onAnalyticsEvent(event: ProfileAnalyticsEvent) {
    when (event) {
        ProfileAnalyticsEvent.PROFILE_OPENED -> trackProfileOpened()
        ProfileAnalyticsEvent.PROFILE_EDITED -> trackProfileEdited()
        ProfileAnalyticsEvent.PROFILE_SAVED -> trackProfileSaved()
        else -> trackProfileDeleted()
    }
}
```
- 위 코드에서 새로운 이벤트 `PROFILE_DELETED`가 추가됐다면, 이 이벤트를 놓치고 else로 인해 `trackProfileDeleted`이 호출될 것.
- 가능한 모든 값을 명시적으로 선언하고, `when`을 철저하게 작성하면 이러한 문제를 쉽게 피할 수 있음.

```kotlin
fun onAnalyticsEvent(event: ProfileAnalyticsEvent) {
    when (event) {
        ProfileAnalyticsEvent.PROFILE_OPENED -> trackProfileOpened()
        ProfileAnalyticsEvent.PROFILE_EDITED -> trackProfileEdited()
        ProfileAnalyticsEvent.PROFILE_SAVED -> trackProfileSaved()
        ProfileAnalyticsEvent.PROFILE_DELETED -> trackProfileDeleted()
        ProfileAnalyticsEvent.PROFILE_CANCELLED -> trackProfileCancelled()
    }
}
```
