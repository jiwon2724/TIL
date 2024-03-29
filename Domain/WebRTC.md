# WebRTC란?

- WebRTC는 끊임없이 연결이 유지되어 요청을 처리하는 상태유지(stateful)프로토콜 입니다.
    - 일반적으로 HTTP 통신은 비상태(stateless) 프로토콜은 일회성으로 요청을 처리하고 응답합니다.
    - 미디어 스트리밍을 제공하는 서버는 일반적인 시스템 구성과는 다른 전략이 필요합니다.

# WebRTC 기술에 대한 이해

- WebRTC는 Web Real-Time Communication의 약어로 웹에서 실시간 미디어(오디오, 비디오)와 데이터 통신을 위해 제안된 기술입니다.
- 이는 유저가 어떠한 네트워크 환경에 있더라도 최대 수백 밀리초(ms)내에 상호작용이 일어날 수 있도록 설계되었습니다
- WebRTC는 모든 네트워크에서 실시간 저지연성을 실현시키기 위해 `RTP`, `ICE` 기반 기술들을 구현하고 있습니다.

> 저지연성이란?
데이터가 발생한 시점부터 처리되어 결과가 나타나기까지의 시간이 매우 짧은 상태를 가리킵니다.
> 

# 연결 방식

![image](https://github.com/jiwon2724/TIL/assets/70135188/8ef66409-c84e-4348-8cf9-7cf4943be894)


- WebRTC를 사용하여 두 사용자는 어떻게 연결되어 있을까요?
- WebRTC는 다양한 프로토콜(ICE, RTP/RTCP, DTLS 등)이 복합적으로 연계되어 있는 양방향 연결인 PeerConnection을 통해 두 사용자를 직접적으로 연결하고 있습니다.
- 연결된 두 사용자는 PeerConnection을 통해 오디오나 비디오 등의 미디어롤 전송하고, DataChannel을 생성하여 미디어 외 데이터를 전송할 수 있습니다.
- 최초 두 사용자를 PeerConnection으로 연결시키기 위해서는 IP, 포트, 미디어 코덱 등의 세션정보를 교환해야합니다.
- 두 사용자 사이에서 PeerConnection을 맺기 위한 세션정보를 교환해주는 서버인 `시그널링 서버`의 도움을 받아 두 사용자가 세션정보를 교환할 수 있습니다.
- 세션을 교환하는 기술에 대한 표준은 존재하지 않으며 WebSocket, STOMP, gRPC 등 다양한 방식으로 구현되고 있습니다.
- 시그널링 서버를 통해 PeerConnection을 맺는 과정은 일반적으로 Offer/Answer 모델에 따라 이뤄집니다.

## Offer/Answer 모델

- Offer/Answer 모델은 SDP를 사용하는 모델입니다.
- 프로세스는 시그널링 서버에서 진행되고, 진행과정은 다음과 같습니다.
    1. Alice가 Bob에게 자신의 IP, 포트, 지원 가능한 미디어 코덱 등의 세션 정보를 시그널링 서버를 통해 전달(Offer)합니다.
    2. Bob은 전달받은 Alice의 세션정보를 바탕으로 PeerConnection을 맺을 준비를 합니다.
    3. Bob은 연결 가능한 자신의 IP, 포트, 미디어 코덱 등의 세션정보를 시그널링 서버를 통해 Alice에게 전달(Answer)합니다.
    4. Alice는 전달받은 Bob의 세션정보를 바탕으로 PeerConnection을 맺습니다.
    5. Alice와 Bob은 PeerConnection을 통해 서로에게 미디어를 전송할 수 있습니다.

# 용어정리

## PeerConnection

- 두 개의 서로 다른 컴퓨터에 돌아가는 Application을 P2P 방식으로 연결해주는 것 입니다.
- 즉, 두 피어(Peer)간 직접적인 연결을 의미합니다.

## P2P(Peer-to-Peer)

- 인터넷에 연결된 다수의 개별 유저들이 중개 기관을 거치지 않고 직접 데이터를 주고받는 것을 의미합니다.
- Peer는 동료라는 뜻으로, 네트워크에 연결된 모든 컴퓨터들이 서로 대등한 동료의 입장에서 데이터나 주변장치 등을 공유할 수 있는 의미입니다.
- 즉, 피어간 데이터나 리소스를 공유하고 교환하는 네트워크 아키텍처입니다.

## 시그널링 서버(Signaling Server)

- 피어(Peer)간 메타데이터를 교환하는데 사용되는 중앙 서버 입니다.
    - 여기서 메타데이터는 다음을 의미합니다.
        - 네트워크 구성 정보
        - 세션 설명 프로토콜(SDP) 등
- 네트워크에서 피어간 직접 연결을 설정하기 위해서 먼저 서로를 찾고, 연결 가능성을 확인하며, 네트워크 구성과 관련된 정보를 교환해야 합니다. 이러한 과정을 `시그널링` 이라고 하며, 시그널링 서버는 이 과정을 중재합니다.

## SDP(Session Decription Protocol)

```kotlin
v=0 // SDP 버전
o=jdoe 2890844526 2890842807 IN IP4 10.47.16.5 // 소유자/생성자 및 세셙 ID
s=SDP Seminar // 세션 이름
i=A Seminar on the session description protocol 
t=2873397496 2873404696 // 세션 활성 시간
a=recvonly
m=audio 49170 RTP/AVP 0 // 미디어 설명
c=IN IP4 10.47.16.5 // 연결 정보
m=video 51372 RTP/AVP 99
a=rtpmap:99 h263-1998/90000 // 속성(코덱 정보 등)
```

- 피어(Peer)간 멀티 미디어 통신 세션을 설정하고 협상하기 위해 사용되는 포맷입니다.
- 이는 통신 세션에 대한 중요한 정보를 담고 있는 문자열이고, 다음과 같은 정보를 교환하는데 사용됩니다.
    - 피어 간 미디어 형식
    - 네트워크 정보
    - 코덱 설정
    - 등
- SDP는 WebRTC 세션의 성공적인 협상과 설정에 있어 핵심적인 역할을 합니다. 피어간에 호환 가능한 미디어 스트림을 교환하고, 효울적인 통신을 위한 네트워크 정보를 설정하는데 필요한 정보를 제공합니다.

## ICE(Interactive Connectivity Establishment)

- 피어 간에 직접적인 연결을 설정하는 프로세스를 용이하게 하는 프레임워크 입니다.
    - 즉, 두 단말이 서로 통신할 수 있는 최적의 경로를 찾을 수 있도록 도와주는 프레임워크
- ICE를 사용하는 이유는 다음과 같습니다.
    - 모든 단말은 각자 환경이 다양하기에 Peer A에서 Peer B까지 단순하게 연결되지 않습니다.
    - NAT(네트워크 주소 변환)이나 방화벽 같은 중간 네트워크 장비를 통과해야 하는 환경에서 두 피어가 서로를 찾고, 통신할 수 있는 경로를 결정하는데 중요한 역할을 합니다.
- 최적의 경로를 찾는 방법은 TURN, STUN 서버를 사용합니다.

## STUN(Session **Traversal Utilities for NAT**)서버

- 주로 NAT뒤에 위치한 장치가 자신의 공용 IP 주소와 포트 번호를 발견할 수 있도록 돕습니다.
    - 해당 정보는 이후 다른 피어와 직접적인 통신을 설정하는데 사용됩니다.
- 클라이언트는 STUN 서버에 요청을 보내고, 서버는 클라이언트의 IP 주소와 포트를 응답합니다.
    - 이 정보를 통해 피어 간 직접 연결을 시도할 수 있습니다.
- 모든 유형의 NAT 시나리오에서 작동하는 것은 아닙니다.

## TURN(Traversal Using Relays around NAT)서버

- STUN 서버로 해결할 수 없는 상황에서 피어 간의 통신을 가능하게 합니다.
- 이는 모든 트래픽을 자신을 통해 중계함으로써, 어떠한 네트워크 제약 조건에서도 피어 간의 통신을 보장합니다.
    - NAT, 방화벽 뒤에 있는 피어도 서로 통신할 수 있습니다.
- 리소스를 많이 사용하므로, STUN을 통한 연결이 선호됩니다. 그러나, 통신이 반드시 성공해야 하는 경우에 TURN을 사용하는 것이 보장된 방법입니다.

## RTP(Real-time Transport Protocol)

- 실시간 음성과 영상 및 데이터를 네트워크로 전송하는 표준 프로토콜입니다.
    - 전화, WebRTC, TV서비스 등에서 사용됩니다.
- 빠른 전송 기능을 지원하기 위해 UDP 프로토콜 위에서 구현되어 있습니다.
    - UDP의 특징인 데이터그램의 분실이나 도착 순서 변경 등의 오류를 RTP에서 해결하는 구조로 이루어져 있습니다.
- RTCP(Real-time Control Protocol)을 이용하여 데이터의 전달 상황을 감시 및 최소한의 제어 기능과 미디어 식별 등을 제공하고 있습니다.
    - RTCP의 사용은 옵션입니다.

## DTLS

# 출처

https://hyperconnect.github.io/2022/12/30/introduction-to-media-server.html

https://medium.com/@asappstudio/webrtc-native-apis-601cf078c3ad
