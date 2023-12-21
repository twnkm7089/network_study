# 네트워크 보안 2

## SSL

### SSL과 TCP/IP
- 일반적인 TCP/IP 스택과 달리, SSL이 적용된 application의 경우 `application layer와 TCP layer 사이에 SSL이 들어간다`.
- 일반적으로 http는 TCP에 의존적이고, `TCP에서 제공하는 기능만 사용 가능`하다.
    - 하지만 TCP는 `보안을 제공해주지 않는다`. 그것이 왜 wireshark 같은 것을 쓸 때 그냥 정보가 모두 보이는지다.
- 그래서 `SSL`이 추가되었다. `새로운 계층은 아니고`, `application layer 내부`에 속해 있고, `라이브러리`로, http와 TCP 사이에 위치한 `인터페이스`다. TCP를 사용하되, 그 전에 SSL로 암호화와 같은 `보안 처리`를 해준 것이다.
    - `Secure Socket Layer`의 줄임말이다.
    - 요즘은 `TLS(Transport Layer Security)`라는 용어를 더 많이 쓴다.
- 이렇게 http message를 SSL을 거쳐 내려보내면 `https`가 된다.
    - http + ssl => https

## 동작 원리
- 크게 handshake -> key derivation -> data transfer -> connection closure로 이루어진다.
- handshake : SSL 통신을 위한 `기본 시작 동작`
    - SSL도 결국 TCP connection 위에서 일어난다. 즉, `TCP connection이 미리 생성`되어 있어야 한다.
    - TCP connection이 `연결된 이후`, client는 server에 SSL 통신을 하고 싶다고 `요청`한다.
    - server는 client에 자신의 `public key 인증서`를 넣어 응답한다.
    - 이를 받고 이를 토대로 `server의 public key`를 알게된다. 그 후, 이를 이용해 `암호화된 메시지`를 보낸다. 물론, 처음에 암호화된 `Symmetric Key`를 보내고 그 후, 이를 바탕으로 통신한다.
    - 참고로, 이 Symmetric key를 그냥 쓰지 않고, 이를 바탕으로 `4가지 key`를 만들어낸다. 이 function은 널리 퍼져 있어, symmetric key만 알면 이 key들을 만들 수 있다.
    - 각각의 역할은 client -> server의 `data 암호화`, client -> server과정에서 `integrity check를 위한 Message Authentication Code(MAC)`, server->client `암호화`, server -> client 의 `MAC`이다. 따로 사용하는 이유는 `key 유출 시의 문제 최소화`를 위해서다.
    - 우리나라 금융기관은 이상하게도 `우리(client)가 인증서를 제공한다`. server(은행)가 편하지만 이상하다. 만약 피싱에 의해 `은행인척하는 사이트에 잘못 보내면 큰일`난다. 정책적인 문제에 의한 결과다.
- data records : 아까 만든 4가지 key를 바탕으로 message가 들어오면 `SSL record`라는 전송형태로 바꾼다.
    - 이 record는 `length, data, MAC`으로 구성된다.
    - 여기서의 `MAC`은 link layer의 MAC이 아닌 `Message Authentication Code`다. `data와 key를 붙인 후, hash function에 집어 넣은 값`이다. key를 붙였으니, attacker는 중간에 이를 수정할 수 없다.
    - 이렇게 `암호화된 메시지`가 보내진다.
    - 이제 TCP segment의 data부분에는 `message 대신 SSL record가 들어간다`. 이렇게 TCP segment의 data부분이 암호화된다. attacker는 메시지를 `읽을 수도 없고, 수정도 불가능`하다. 
    - 이제 attacker가 할 수 있는 것은 여러개로 분할된 TCP segment에 들어있는 `data의 순서를 바꿔치기`하는 정도다.
        - 이 문제를 해결하기 위해 `SSL record 안에도 순서를 넣어준다`.
    - 또, 중간에 `FIN segment를 보내 강제 종료` 시킬 수도 있다.
        - 이 문제 해결을 위해 SSL record에 `type field(1bit)`를 넣어, `data는 1, 통신 끝내기는 0`을 입력한다. 그 후, 이것까지 포함해 만든 hash값으로 MAC을 만든다.
- 운영하는 서버에 https를 원하면 `인증기관에 가서 인증서를 받아오자`.
- 실제 SSL은 `어느 종류의 암호화 알고리즘 쓸지 negotiation`하는 과정도 있다.

## 정리
- data가 여러개의 data fragment로 `분할`된 후, 이를 바탕으로 `MAC`이 붙고, 이 data와 MAC을 합쳐 `암호화`해서 `TCP segment의 data`로 변환된다.

## Firewalls
- 하나의 네트워크의 `gateway`에 들어가 있어, `외부로 나가거나 들어가는 패킷을 감시`한다. 그래서 이 패킷을 통과시킬지, 필터링시킬지 `monitoring하는 device`다.
    - 대부분의 network 기기에는 설치되어있다.
    - 어떤 패킷 통과시키고, 어떤 것은 안시킬지는 `운영자의 정책에 따라 결정`된다. 
    - 예를 들면, `no outside web access` 같은 엄청난 것도 port 80의 바깥으로 나가는 모든 packet을 drop시키라는 식으로 할 수 있다.
        - 근데, 이를 위해서는 `transport layer`를 보아야 한다. router는 Network layer 기기이니, `layer violation`이 일어난다.
        - 하지만, 이런 룰을 집어넣으면 `기술적으로는 가능하나, 사람들이 엄청 반발할 것`이다. 네트워크 관리자 입장에서는 `최대한 막는 보수적 방법이 좋긴하나, 사용자 만족을 위해 이런 것은 쉽지 않다`.
        - 참고로, `ssh protocol을 이용하는 서버는 port 22`다. 이게 막혀 있으면 화가 나겠죠? 이런식으로 말도 안되는 정책으로 인해 막힌게 있다면 적극적으로 항의합시다.
    - 그 외에도 이 네트워크 내부에서 `유일하게 운영 가능`한 웹 서버를 지정하는 등 다양한 방법이 있다.
- firewall안에 들어가는 rule table이 필요하다. 이게 `Access Control Lists`다.
    - `위에 있는 것일수록 우선순위`를 가지고, `위에서부터 보며 지금의 패킷이 해당되는 경우 이에 대한 설정된 내용(allow, deny)`를 수행한다.
    - 근데, TCP protocol을 사용하는 packet을 주고받으려면 TCP connection이 있어야 한다. `packet만 보면 TCP connection이 없었음에도 통과시키는 경우`가 있다.
    - 그래서 발전된 경우, `이에 해당하는 TCP connection이 있는지도 검사`한다.