> `gradle/wrapper/gradle-wrapper.jar does not exist! Needs to be committed.`
> 

## .jar 파일 .gitignore에서 제외하기

- .jar 파일을 제외하게 되면 gradle wrapper를 찾지 못합니다.
- .gitignore에서 .jar 확장자가 추가되어 있다면, 이를 수정하고 gradle-wrapper.jar 파일을 추가해줘야 합니다.

```kotlin
git add gradle/wrapper/gradle-wrapper.jar
git commit -m "input your commit message"
```

## public 라이브러리로 활용한다면 접근권한 풀어주기
| Received status code 401 from server: Unauthorized
- 라이브러리를 dependencies에 추가할 때, 위 같은 에러가 나온다면 jitpack 홈페이지에서 다음과 같이 설정해줘야 합니다.
<img width="590" alt="image" src="https://github.com/jiwon2724/TIL/assets/70135188/aae17e64-95b8-4013-990a-d40954265cd2">

- 톱니바퀴 버튼 클릭 후 Everyone 섹션에 자물쇠 off로 설정하면 됩니다.
