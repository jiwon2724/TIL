# Junit
- 자바의 표준 단위 테스트 프레임워크입니다.
- JUnit4와 JUnit5(JUnit Jupiter) 버전이 있으며, JUnit5는 모듈화된 구조와 확장성 있는 기능을 제공합니다.

### 주요 기능
- 어노테이션 기반 테스트: `@Test`, `@Before`, `@After`, `@BeforeClass`, `@AfterClass` 등 어노테이션을 통해 테스트 메서드를 정의합니다.
- 다양한 Assertion 메서드: 다양한 Assertion한 메서드를 통해 테스트 결과를 검증합니다.
- 호환성: 다양한 빌드 툴(Maven, Gradle)과 통합되고, 여러 IDE에서 지원됩니다.

### JUnit 테스트 코드
```kotlin
// 테스트 대상 클래스
class StringUtil {
    fun reverse(input: String): String {
        return input.reversed()
    }

    fun isPalindrome(input: String): Boolean {
        val reversed = input.reversed()
        return input.equals(reversed, ignoreCase = true)
    }
}

// JUnit 테스트 클래스
class StringUtilTest {
    private val stringUtil = StringUtil()

    @Test
    fun testReverse() {
        val result = stringUtil.reverse("hello")
        assertEquals("olleh", result)
    }

    @Test
    fun testIsPalindrome() {
        assertTrue(stringUtil.isPalindrome("madam"))
        assertFalse(stringUtil.isPalindrome("hello"))
    }
}

```
