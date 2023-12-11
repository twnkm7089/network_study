# 링크 계층 3

## 시나리오
- A가 B에게 패킷을 보내고자 한다. 이 둘은 서로 다른 subnet에 속해 있다. 중간에 R이라는 router가 있고, 양쪽 subnet에 동시에 속해 있다.
- A가 B에게 패킷을 보내려 한다. destination IP는 B다.
- forwarding table을 보고 B로 가기 위한 다음 hop의 IP 주소를 찾는다. 여기서는 gateway router인 R의 IP address가 적혀 있다.
- ARP table을 이용해 이 address에 해당하는 MAC address를 찾는다.
- CSMA/CD를 이용해 일단 listen후, 아무것도 감지 안되면 보낸다.
- router는 data를 빼내서, destination이 B인 것을 알아낸다. router에서 B로 가기 위한 IP address를 forwarding table에서 참조하고, 해당 주소로 가기 위한 MAC address를 ARP table에서 찾고, 마지막으로 CSMA/CD를 통해 듣고, 전송한다.

### 질문
- forwarding table을 만들다 보면 `ARP table도 자연스럽게` 만들어지지 않나요?
    - 답 : `그렇다.` interaction하다보면 자연스럽게 작성된다. 다만, 한번 적힌 것이어도 TTL에 의해 유효 시간이 있고, 계속 작성해야 한다.

## Switch
- 과거에는 coaxial cable을 이용한 bus topology를 사용했다.
- 이제는 switch를 이용한 `star topology`를 사용한다. 중간에 switch라는 device를 두고, 새로운 node가 필요하면, `switch에 꽂아서 연결`한다.
    - 또, switch는 `collision domain`을 분리한다. collision domain은 동시에 말할 경우 충돌이 일어날 수 있는 domain이다. switch가 중간에서 host를 분리시켜 주니 다른 신호의 전파를 차단한다.
    - 또, switch는 `host입장에서는 보내지지 않는다`. host입장에서는 그저, ethernet이 연결되어 있을 뿐이다. switch에는 `MAC address도 없고, 무엇도 없다`. 하지만 collision domain은 분리해준다.

### 그러면 무엇이 가능한가? 동시전송
- 동시에 여러 host가 전송하는 것이 가능하다. collision domain이 분리되어 있기 때문이다. A -> A', B -> B'에 동시에 보낼 수 있다.
    - 만약 둘 다 A'으로 보내고 싶었다면? 둘 중 하나를 먼저 보내주고, 나머지는 나중에 보내준다.
- 근데, 그러러면 `어디에 어느 host가 붙어 있는지 알아야` 한다. 이는 switch 내부에 `switch table`이 있어, 이를 이용한다. 어느 interface에 어느 MAC address를 가진 host가 붙어 있는지 저장되어 있다.
- 그럼, switch table은 어떻게 만들어 지는가? `self-learning`.
    - 예를 들어, A->A'으로 가는 frame이 도착한다. switch는 최소한, `A가 어느 port에 연결되어 있는지 알게 된다`(이 패킷이 들어온 포트에 연결되어 있을 것이므로).
    - 근데, A'에 대한 정보는 아직 없다. 그래서 `flooding`을 한다. `연결된 나머지 모든 포트에 보내준다`.
    - 나중에 해당 위치에 대한 정보가 있으면 `selectively send`한다. 즉, 정보가 있으면 그쪽으로만 보낸다.
    - 또, 웬만한 packet은 gateway router로 향한다. 고로, switch에는 gateway router과 관련된 정보는 아마 이미 self-learning 되어 있을 것이다.
- `switch끼리 연결`시켜서 `switch를 계층화` 시켜 `더 큰 LAN`을 만들 수도 있다. 여러 switch가 보이더래도 host입장에서는 `switch는 없는 것`이므로, `network layer 관점에서는 그냥 하나의 네트워크`다.
- switch가 여러개 있다. A -> S1 -> S2 -> S3 -> I의 루트를 타야 한다. 이 때는 어떻게 해야 할까?
    - `추가로 무엇을 할 필요 없다.` `self learning`이다. 자연스럽게 어떤 host로부터 온 정보가 어느 port로 들어오는지가 기록되니 정보는 쌓이고, 정보가 없어도 flooding으로 적절한 위치에 도착된다.

### 질문
- switch vs hub vs 가정용 공유기
    - hub는 `완전 다른 용어`. hub는 `physical layer device`.
    - switch vs 가정용 공유기
        - 가정용 공유기 : 회사에서 회선을 하나 주고, 이게 연결이 되어 있다. 이 cable은 이더넷 케이블이 아니라 회사에서 제공하는 특정 케이블이다. 원래 이 케이블에 컴퓨터 하나만을 연결해야 하는데, 그 대신 공유기를 연결해 여럿이 쓰게 한다.
            - 공유기는 `application layer까지 모두 존재하는 특수한 컴퓨터`다. 여기서는 `NAT를 생성`해 `내부 IP를 만들어 DHCP를 통해 다른 기기에 뿌린다`. `DNS`도 여기서 동작한다.
            - 이는 여러 기기가 하나의 `gateway router에 연결되어 있는 모습`이다. 이게 공유기의 역할이다. 다만, 대형 네트워크와 다르게 통신사의 modem과 연결되고, 통신사의 네트워크와 연결될 뿐이다.
        - switch : `단순히 여러 host를 엮어준 것`이다. 네트워크 상에는 그냥 여러 host가 성형으로 엮인 모습일 뿐이다.
        - 공유기에 공유기를 연결하면? 계층화된 네트워크가 나타난다. 스위치에 스위치를 연결하면? 그냥 큰 네트워크다.
        - 스위치에는 MAC address가 없다. routing을 할 때도, switch의 위치는 적히지 않는다.
        - router을 추가하면 하나의 서브넷을 만들어낸다. switch는 그런거 없다.
- 네트워크 확장 시 router를 쓸까, switch를 쓸까... 알아서 결정해라.
- gateway router에서 DHCP 서버가 동작한다.

- 그냥 router vs switch
    - router : network계층까지 지원
    - switch : link계층까지 지원

## 데이터 센터 네트워크
- 대용량의 host는 어떻게 처리할까?
- 서버가 여러개 돌아가고 있어서, 구글로 들어온 패킷이라 가정할 때, 서버 하나가 아닌 load가 제일 적게 걸려있는 곳에서 해당 패킷을 처리하게 보낸다.
- 이 데이터 센터는 매우 크다.
- 이 서버들은 `switch`로 연결되어 있다. switch가 계층화되어 여러 서버를 연결하는 구조다. 외부의 request가 오면 `load balance` 되어 load가 제일 적은 server를 찾아 해당 위치로 이동하여 처리한다.