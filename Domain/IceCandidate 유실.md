# IceCandidate 유실
> 시그널링 과정을 위해 소켓으로 통신하던 도중 다음과 같은 상황이 발생했습니다. createOffer호출 후 setLocalDescription을 호출하여 candidate가 보내지는데, WebRTC 통신
> 자체가 비동기로 진행이 되다보니 createOffer보다 candiate가 1~2개씩 먼저 도착하는 상황이 있습니다.

## IceCandidate가 유실된다면?
- 10번 시그널링을 시도하면 8번은 성공하지만, 2번정도는 간혈적으로 화상 및 음성통화가 되지 않았습니다.
- 많은 candidate중에 중요한 candidate가 유실되어 연결이 안된것이라 추측됩니다.


## 해결방법
1. Thread.sleep을 통하여 Thread Block시키기
   - 우선적으로 떠오르는건 Thread Block시키는 방법이지만, 좋은 방법은 아니라 테스트만 진행했습니다.
   - thread { .. } 블록을 만들어 setLocalDescription 함수를 호출하기전 Thread.sleep(500L)을 넣어주니 유실은 일어나지 않았습니다.
   - 유실이 일어나지 않은 것을 확인하고, 이를 코루틴을 사용하여 해결할 수 있다고 생각했습니다.
3. 코루틴을 사용하여 순차적 코드로 만들어주기
   - 기존엔 CoroutineScope(Dispatchers.IO).launch { ... } 코드가 없었습니다.
   - 그래서 `sendOffer`, `setLocalDescription`도 일반함수입니다.
   - 현재는 일시 중단 함수입니다.
   - `sendOffer`, `setLocalDescription`를 일시 중단 함수로 만들어주면서, 해당 코루틴 스코프내에서 순차적으로 동작하게 만들었습니다.
```kotlin
with(peerConnection) {
    createOffer(object : SdpObserver {
        override fun onCreateSuccess(sessionDescription: SessionDescription)  {
            CoroutineScope(Dispatchers.IO).launch {
                Log.d(RtcClient.TAG, "createOffer onCreateSuccess")
                sendOffer(
                    sessionDescription = sessionDescription,
                    channelId = channelId,
                    targetUserId = targetUserId
                )
                setLocalDescription(sessionDescription)
                Log.d(TAG, sessionDescription.description.toByteArray().size.toString())
            }
        }

        override fun onSetSuccess() {
            Log.d(RtcClient.TAG, "createOffer onCreateFailure")
        }

        override fun onCreateFailure(reason: String) {
            Log.d(TAG, "createOffer onCreateFailure")
        }

        override fun onSetFailure(reason: String) {
            Log.d(TAG, "createOffer onSetFailure")
        }
    }, mediaConstraints)
}
```
3. candidate ArrayList<IceCandidate>에 저장하기
   - 동기식, 비동기식 신경안쓰고, setLocalDescription 호출이 끝난다음 sendCandidate를 함수를 실행하는 것 대신 ArrayList에 add하고 peer가 들어왔을 때(offerReceive)보내주는 방법입니다.
   - WebRTC가 P2P 방식이다보니 해당 방법이 제일 효과적이라고 생각했습니다. (같이 개발하는 팀원분의 아이디어입니다^^7)
   - fcm을 사용하여 push notification이 발생했을 때, payload로 들어온 data의 유저의 정보(targetUserId, sdp 등)를 가지고 소켓에 connect 됐을 때 offerReceive를 호출하면 됩니다.
