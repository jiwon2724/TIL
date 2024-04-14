# PendingIntent

> 기본적으로 일반적인 Intent의 "지연된" 또는 "예약된" 실행을 가능하게 합니다. 이를 통해 다른 앱이나 안드로이드 시스템이 애플리케이션이 실행 중이지 않을 때에도 특정 작업을 대신 실행할 수 있도록 합니다.
> 
- 다른 앱이나 시스템에 의해 나중에 실행될 수 있는 권한을 부여하는 것이 가능합니다.
    - ex) A앱 실행중 B앱의 컴포넌트(Activity, Service등)가 실행될 수 있음.
- Intent를 당장 수행하지 않습니다. 특정 시점에 수행하며, 이는 보통 앱이 구동되고 있지 않을 때를 의미합니다.

## Intent vs PendingIntent

- `Intent` : 현재 사용중인 앱에서 컴포넌트간 메세지 전달 방식으로 사용됩니다. 즉, A라는 앱을 사용중이라면 A앱에서만 통신이 가능합니다.
- `PendingIntent` : 특정 시점에(앱이 구동되고 있지 않을 때 등) Intent가 수행될 수 있게 보장합니다.
    - 다른 애플리케이션의 컴포넌트에게 나의 애플리케이션을 특정 시점에 `Intent`를 실행할 권한을 부여합니다.

## 사용방법

```kotlin
val requestCode: Int = Random().nextInt()
val intent: Intent = Intent(this, MainActivity::class.java)
val pendingIntent: PendingIntent = PendingIntent.getActivity(
    this,
    requestCode,
    intent,
    PendingIntent.FLAG_UPDATE_CURRENT
)
```

- `PendingIntent` 를 참조하여 다음과 같은 컴포넌트를 호출할 수 있습니다. 메서드 이름에서 유추할 수 있듯이 다음과 같은 기능을 호출합니다.
    - getActivity(Context, int, Intent, int)
        - `Intent.FLAG_ACTIVITY_NEW_TASK` 를 사용하여 호출해야합니다.
    - getService(Context, int, Intent, int)
    - getBroadcast(Context, int, Intent, int)
    - getForegroundService(Context, int, Intent, int)

## 여러가지 Flags

> PendingIntent의 flag는 PendingIntent의 생성 및 재사용 방식을 결정합니다.
> 
1. **FLAG_ONE_SHOT**: 이 플래그는 `PendingIntent`가 한 번 사용된 후에는 더 이상 사용될 수 없음을 나타냅니다. 사용 후에는 자동으로 취소됩니다.
2. **FLAG_NO_CREATE**: 이 플래그가 설정되면, `PendingIntent`를 생성하지 않고 기존의 `PendingIntent`를 검색만 합니다. 만약 해당하는 `PendingIntent`가 이미 존재하지 않으면 `null`을 반환합니다.
3. **FLAG_CANCEL_CURRENT**: 이 플래그가 설정되면, 기존에 존재하는 `PendingIntent`를 취소하고 새로운 것을 생성합니다. 이는 기존의 데이터를 덮어쓰고 싶을 때 유용합니다.
4. **FLAG_UPDATE_CURRENT**: 기존에 존재하는 `PendingIntent`가 있을 경우, 그 `PendingIntent`의 내용을 새로운 `Intent`의 내용으로 업데이트합니다. 이 플래그는 데이터를 최신 상태로 유지하고자 할 때 자주 사용됩니다.

## Android12 대응

> Android 12부터는 모든 PendingIntent에 FLAG_IMMUTABLE 또는 FLAG_MUTABLE 중 하나를 명시적으로 설정해야 합니다.
> 
- FLAG_MUTABLE **:** PendingIntent가 변경 가능함을 명시적으로 표시하게 됩니다.
    - PendingIntent가 변경 가능함을 명시적으로 표시하게 되며, 이 경우 개발자는 PendingIntent의 사용 방법을 신중히 고려해야 합니다.
- FLAG_IMMUTABLE : PendingIntent가 불변임을 명시적으로 표시하게 됩니다.

```kotlin
if (android.os.Build.VERSION.SDK_INT >= 23) {
  // FLAG_IMMUTABLE을 사용하여 PendingIntent 생성
    val intent = Intent(this, ExampleActivity::class.java)
    val pendingIntent = PendingIntent.getActivity(
        this,
        0,
        intent,
        PendingIntent.FLAG_ONE_SHOT or PendingIntent.FLAG_IMMUTABLE
    )
} else {
  // PendingIntent를 생성하는 기존 코드
}
```

- `ExampleActivity`를 실행하기 위한 `PendingIntent`를 생성하며, PendingIntent가 단 한 번만 사용될 수 있고 (`FLAG_ONE_SHOT`), 생성 후에는 수정될 수 없음 (`FLAG_IMMUTABLE`)을 보장합니다.

## **암시적 PendingIntent** 보안상의 문제

> PendingIntent 를 암시적으로 사용하는 경우, 다음 사항 중 일부나 모두를 적용하여 취약점을 해결하는 것이 좋습니다.
> 
1. 기본 인텐트의 작업, 패키지 **및** 구성요소 필드가 설정되었는지 확인합니다.

```kotlin
// 암시적 기본 인텐트를 생성하고 PendingIntent에서 래핑
Intent base = new Intent("ACTION_FOO");
base.setPackage("some_package");
PendingIntent pi = PendingIntent.getService(this, 0, base, 0);
```

2. PendingIntent가 신뢰할 수 있는 구성요소에만 전달되는지 확인합니다.
3. `FLAG_IMMUTABLE`(SDK 23에 추가됨)을 사용하여 PendingIntent를 생성합니다. 앱이 SDK 22 이하를 실행하는 기기에서도 실행되는 경우 PendingIntent 생성을 강화하면서 이전 옵션을 적용하는 것이 좋습니다.
