# Hilt 내부동작

## Annotation

- 소스코드를 해치지 않으면서, 컴파일러에게 부가정보를 제공하는 기능.
- 클래스, 필드, 메서드 및 기타 요소에 선택적으로 선언이 가능함.
- 런타임에서 참조가 가능함.

## Annotation 프로세서

- `Hilt`는 Annotation 기반으로 동작함.
- Annotation 프로세서는 컴파일 타임에 Annotation을 스캔하고, 소스코드를 검사 또는 생성함.
    - 빌드가 완료된 후 프로젝트 소스코드가 자동으로 생성되는 원리가 Annotation 프로세싱으로 인해 생성된 코드임.
        - 하나의 Annotation을 처리하는 단계를 round라 부름.
            - 몇차례의 round를 거치며 Annotation 프로세서가 코드를 스캔하게됨.
- Hilt에서 사용되는 주요 Anntation의 종류는 다음과 같음.
    - `@HiltAndroidApp`
    - `@AndroidEntryPoint`
    - `@Module`
    - `@InstallIn`
    - `@HiltViewModel`
    - 등

## Hilt Annotation 처리 요약

- Hilt 및 Dagger2는 전용 Annotation을 사용해서 컴파일 타임에 Annotation을 적절히 처리함.
- 컴파일 타임에 오브젝트 그래프에 문제가 있다면, 예외를 발생시키고 빌드를 중단시킴.
    - 컴파일 타임에 검사하므로, 개발과 테스트 부분에서 시간 단축에 용이하고 안전함.
    - 오브젝트 그래프는 객체들 간 종속성 관계를 나타내는 구조이며, Hilt, Dagger2에선 이 그래프를 자동으로 생성되고, 의존성 주입하는 데 사용됨.
    - 오브젝트 그래프를 그림으로 보면 다음과 같음.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/17b7261d-8574-4fae-97a1-5f7367a227bc/2bedea29-4ebc-4093-8bf9-a9cb0627a429/Untitled.png)

- 생성된 소스코드를 기반으로 동작하므로, 리플렉션을 사용하지 않아도 됨.
    - 리플렉션은 런타임에서 프로그램 구조를 검사하고 조작할 수 있는 기능임.
        - ex) 애플리케이션 내의 클래스, 메소드, 변수 등에 대한 정보를 조회하거나 수정 가능함.
    - 리플렉션은 런타임에 타입 체크와 메소드 호출을 처리하기 때문에 일반적인 코드 접근 방식보다 느림.

## 바이트 코드 변조

- 바이트 코드란 자바 소스코드가 컴파일을 거쳐 나온 결과물임.
    - 자바나 코틀린으로 소스코드를 작성하고, 컴파일러를 통해 JVM이 해석할 수 있는 산출물임.
- 바이트 코드 변조란 앱 빌드 과정에서 컴파일러가 바이트 코드를 산출하면 이를 변조하는 프로세스임.
    - 바이트 코드 변조를 사용하면, 소스코드를 해치지 않으면서 Hilt 사용시 편리하게 의존성 주입을 사용할 수 있도록 도와줌.
- Dagger2에는 없고, Hilt에 새로 추가된 기능임.

## 바이트 코드 변조 예시

```kotlin
@HiltAndroidApp
public class App extends Application()
```

- 위 코드에서 Annotation 프로세싱을 거치면 Hilt prefix가 붙은 Hilt_App 클래스를 생성하게됨.

```mermaid
graph LR
  App --> 컴파일러 --> Hilt_App
```

- 의존성 주입을 하기 위해 Annotation 프로세싱을 거쳐 나온 Hilt_App 클래스를 반드시 참조해야함.

```kotlin
@HiltAndroidApp
public class App extends Hilt_App()
```

- 소스코드는 위 처럼 바뀌어야함.
- 하지만 위 처럼 코드를 작성하게되면 몇가지 불편한 점이 있음.
    - Hilt_App이 생성되기 이전이라면 사용할 수 없음.
    - Hilt는 클래스 이름을 사용해서 prefix다음 클래스 이름을 붙이므로, 클래스 이름을 바꾸면, `Hilt_App` 이름도 바뀌게됨.
- 위 같은 불편함을 지원하고자 `Hilt_XXX` 를 상속하도록 바이트 코드를 변조함.

```mermaid
graph LR
  App --> Hilt_App --> Application
```

- 바이트 코드 변조를 사용하면, 소스 코드 상에서 Application 상속을 유지하며, 컴파일 이후에 바이트 코드 내부에는 바이트 코드 변조를 통해 위 처럼 의존관계가 형성됨.
    - 위 예시는 App 클래스에서 `@HiltAndroidApp`을 사용한 것임.
- 즉, 코틀린 or 자바 컴파일러가 컴파일 타임에 바이트 코드를 산출하고, 산출된 코드로 바이트 코드 변조를 하여 Hilt_XXX를 상속하게 되는 것.

## Hilt의 전체 프로세스

- Hilt의 전체 프로세스를 요약한 다이어그램은 다음과 같음.
```mermaid
graph LR
  Hilt_애노테이션이_포함된_소스코드 --> 애노테이션_프로세싱 --> 생성된_소스코드 --> 컴파일 --> 바이트코드_산출 --> 바이트_코드_변조 --> APK/AAB패키징
```
