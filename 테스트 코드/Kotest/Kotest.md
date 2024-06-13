# Kotest
> 테스트 프레임워크 개발 생명 주기 전반에서 품질을 유지할 수 있도록 도움을 주고, 재사용 가능한 코드를 작성하는데 중요한 역할을 합니다.
- Kotlin으로 작성 테스트 프레임워크로, BDD, TDD 스타일의 테스를 쉽게 작성할 수 있도록 도와줍니다.
- 다양한 테스트 스타일을 지원합니다.
  - ex) `StringSpec`, `FunSpec` 등
- 다양한 매칭 기능을 제공하여 여러 조건을 쉽게 검증할 수 있습니다.
  - ex) `shouldBe`, `shouldNotBe`
- 테스트 라이프사이클 관리가 가능합니다.
- 등

## Kotest 시작하기
```kotlin
testImplementation 'io.kotest:kotest-runner-junit5:5.9.1' // 2024.06.08 기준 최신버전
```
- build.gradle에 의존성을 추가해줍니다.

# Kotest 명세 스타일
- 다양한 명세 스타일을 제공하여 테스트를 보다 직관적이고 유연하게 작성할 수 있도록 합니다.
-  각 명세 스타일은 고유의 형식과 문법을 가지고 있으며, 특정한 테스트 시나리오에 맞게 선택할 수 있습니다.

### StringSpec
- 문자열을 사용하여 테스트를 정의하는 간단한 스타일입니다.
- 테스트 케이스는 문자열로 설명되며, 블록 내 테스트 로직이 들어갑니다.
- `shouldBe`는 Kotest에서 제공하는 매처 함수입니다.
  - 기대하는 값과 실제 값을 비교하는데 사용됩니다.

```kotlin
class MyStringSpec : StringSpec({
    "length of 'hello' should be 5" {
        "hello".length shouldBe 5
    }

    "reverse of 'kotlin' should be 'niltok'" {
        "kotlin".reversed() shouldBe "niltok"
    }
})
```
### FunSpec
- 함수 기반의 테스트 스타일입니다.
- 각 테스트 케이스는 `test` 함수 내에 정의됩니다.

```kotlin
class MyFunSpec : FunSpec({
    test("length of 'hello' should be 5") {
        "hello".length shouldBe 5
    }

    test("reverse of 'kotlin' should be 'niltok'") {
        "kotlin".reversed() shouldBe "niltok"
    }
})
```
### DescribeSpec
- BDD(Behavior-Driven Development) 스타일의 테스트를 작성할 때 사용합니다.
- `describe`와 `it` 블록을 사용하여 테스트의 계층 구조를 정의하고, 각 테스트 케이스를 명확하게 설명합니다.
- `describe` 블록은 테스트 그룹을 설명합니다.
   - 해당 블록 내에서 관련된 테스트 케이스들을 정의합니다.
- `it` 블록은 개별 테스트 케이스를 설명합니다.
```kotlin
class MyDescribeSpec : DescribeSpec({
    describe("a string") {
        it("should have length 5 when value is 'hello'") {
            "hello".length shouldBe 5
        }

        it("should be 'niltok' when reversed from 'kotlin'") {
            "kotlin".reversed() shouldBe "niltok"
        }
    }
})
```

### ShouldSpec
- BDD 스타일의 또 다른 형태로, should 키워드를 사용하여 테스트를 작성합니다.
```kotlin
class MyShouldSpec : ShouldSpec({
    should("have length 5 when value is 'hello'") {
        "hello".length shouldBe 5
    }

    should("be 'niltok' when reversed from 'kotlin'") {
        "kotlin".reversed() shouldBe "niltok"
    }
})
```

### BehaviorSpec
- 명확한 BDD 스타일로, Given, When, Then 블록을 사용하여 시나리오를 구성합니다.
```kotlin
class MyBehaviorSpec : BehaviorSpec({
    Given("a string 'hello'") {
        When("its length is checked") {
            Then("it should be 5") {
                "hello".length shouldBe 5
            }
        }
    }

    Given("a string 'kotlin'") {
        When("it is reversed") {
            Then("it should be 'niltok'") {
                "kotlin".reversed() shouldBe "niltok"
            }
        }
    }
})
```
