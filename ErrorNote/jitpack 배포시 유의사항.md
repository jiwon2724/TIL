> `gradle/wrapper/gradle-wrapper.jar does not exist! Needs to be committed.`
> 

## .jar 파일 .gitignore에서 제외하기

- .jar 파일을 제외하게 되면 gradle wrapper를 찾지 못합니다.
- .gitignore에서 .jar 확장자가 추가되어 있다면, 이를 수정하고 gradle-wrapper.jar 파일을 추가해줘야 합니다.

```kotlin
git add gradle/wrapper/gradle-wrapper.jar
git commit -m "input your commit message"
```
