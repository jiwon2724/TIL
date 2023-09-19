# HashMap

- Key를 Value에 연관시키는 구조를 가진 자료구조임.
- 특정 Key의 Value에 대한 빠른 검색, 삽입, 삭제를 가능하게 하는것이 목적임.

### 동작 원리

<img width="624" alt="image" src="https://github.com/jiwon2724/TIL/assets/70135188/12ef48ae-ef08-4b9a-90e3-26128868d4d5">


- Key를 받아서 이를 해쉬 함수를 적용함.
    - 해쉬 함수란 임의의 길이의 데이터를 고정된 길이의 데이터로 매핑하는 함수임.
        - 결과로 얻어지는 값을 hashCode라고 부름.
    - 해쉬 함수를 적용하는 이유는 데이터의 효울적인 저장 및 검색을 하기 위함임.
        - hashCode를 사용하여 데이터를 O(1)의 시간복잡도로 빠르게 저장하거나 검색 가능.
        - 같은 입력값은 항상 같은 hashCode를 생성함. 이는 중복된 키를 저장 방지함.
    - 코틀린 문서에는 HashMap의 put, get연산에 hash함수를 적용하는데, Key값의 hashCode를 가지고 새로운 해쉬 값을 만듦.
    
- 해쉬 함수의 결과 hashCode(해쉬 값)을 가지고 저장될 위치를 결정함.

### 충돌

- 해쉬 함수의 결과로 저장될 위치를 결정할 경우 해시 충돌이 발생할 수 있음.
    - 해시 충돌이란 두 개 이상의 다른 키가 동일한 hashCode를 반환하는 경우를 의미함.
        - HashMap은 이런 해시 충돌을 내부적으로 관리함.
- 충돌 해결 방법엔 분리 연결법과 개방 주소법이 있음.

### 분리 연결법

<img width="568" alt="image" src="https://github.com/jiwon2724/TIL/assets/70135188/0cff5357-e5c6-4f46-8670-b1fd78002395">


- 동일한 데이터에 대해서 자료구조를 활용해 추가 메모리를 사용하여 중복된 데이터의 주소를 저장하는 방식임.

### 개방 주소법

- 추가 적인 메모리를 사용하는 분리 연결법과 다르게 비어 있는 해시 공간을 활용하는 방법임.
- 개방 주소법은 다음과 같이 3가지 방식이 존재함.
    - Linear Probing : 현재 index(hashCode)로부터 고정폭 만큼씩 이동하여 비어 있는 곳에 데이터를 저장함.
    - Quadratic Probing : 저장순서 폭을 제곱으로 저장하는 방식임.
    - Double Hashing Probing : 해시된 값을 한번 더 해싱하여 해시의 규칙성을 없애버리는 방식임.

<img width="631" alt="image" src="https://github.com/jiwon2724/TIL/assets/70135188/f1432f78-4de8-4541-99f2-4df93d31865e">


### 코드 - Kotlin

```kotlin
internal fun addKey(key: K): Int {
        checkIsMutable()
        retry@ while (true) {
            var hash = hash(key)
            // put is allowed to grow maxProbeDistance with some limits (resize hash on reaching limits)
            val tentativeMaxProbeDistance = (maxProbeDistance * 2).coerceAtMost(hashSize / 2)
            var probeDistance = 0
            while (true) {
                val index = hashArray[hash]
                if (index <= 0) { // claim or reuse hash slot
                    if (length >= capacity) {
                        ensureExtraCapacity(1)
                        continue@retry
                    }
                    val putIndex = length++
                    keysArray[putIndex] = key
                    presenceArray[putIndex] = hash
                    hashArray[hash] = putIndex + 1
                    _size++
                    if (probeDistance > maxProbeDistance) maxProbeDistance = probeDistance
                    return putIndex
                }
                if (keysArray[index - 1] == key) {
                    return -index
                }
                if (++probeDistance > tentativeMaxProbeDistance) {
                    rehash(hashSize * 2) // cannot find room even with extra "tentativeMaxProbeDistance" -- grow hash
                    continue@retry
                }
                if (hash-- == 0) hash = hashSize - 1
            }
        }
    }
```

- 위 코드는 HashMap에 Key를 추가하는 코드임.
- 만약 현재 인덱스가 비어있다면(`index <= 0`), 새로운 키를 추가하고, 해당 인덱스에 대한 정보도 업데이트 함.
    - 비어있지 않다면 이는 충돌상황임.
- 만약 현재 인덱스에 이미 동일한 키가 있다면(`keysArray[index - 1] == key`), `-index` 값을 반환하여 중복된 키가 있다는 것을 알림.
    - 키는 중복이 안되므로, 중복된 키가 있는걸 알림.
- 충돌이 난 경우 탐사거리를 늘려가며 위 동작을 반복함.
    - 해시의 사이즈를 늘려주고 retry@ while부터 다시 시작함.
- `if (hash-- == 0) hash = hashSize - 1` 충돌이 났지만, 탐사 거리를 충족하는 경우엔
    - 개방 주소방법인 `Linear Probing` 를 사용하여 탐색 후 비어있는 곳에 데이터를 저장.
