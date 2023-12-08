# 링크 계층 2

## LANs(Local Area Networks)
- 과거 subnet에 대해 배운 기억이 있을 것이다.
- 이 subnet이 LAN으로 구성된 것이다.
- `라우터를 거치지 않고 접근 가능한 host의 집합`.
- 결국 여러 host가 연결된 채널은 하나의 LAN을 이룰 수 있다.

## Ethernet
- 70년대에 나와 지금까지 쓰이는 MAC protocol.

### pysical topology
- bus형(과거)
    - 버스에 여러 host가 연결된 형태.
- star형(현재)
    - 가운데에 `switch`가 있고, 모든 host가 이 switch에 연결되어, 이 switch를 통해 서로 이어진 형태.

### Ethernet frame structure
- IP packet은 link layer의 전송단위인 frame의 데이터 부분으로 들어간다.
- frame도 헤더와 데이터 부분으로 존재한다.
- frame header
    - premeable
    - `destination address/ source address`
    - `type` : `상위 레이어`가 어떤 protocol인지. 위로 올렸을 때 처리해야 하므로. 보통은 IP protocol의 코드를 가지겠죠?
- `tail에 CRC 붙는다.`

### 충돌을 어떻게 해결할 것인가?
- Ethernet의 MAC protocol은 `CSMA/CD`.
- `말하기 전에 듣고`, 충돌이 일어났다면 멈추고 `랜덤한 시간만큼 기다린다(binary backoff)`. 그 후, 재전송한다.

#### TCP 재전송 vs Ethernet 재전송
- TCP 재전송 : TCP segment를 보냈는데, destination으로부터 ACK이 안 왔을 때
- Ethernet 재전송 : 한 hop 사이(destination과 무관)에 collision으로 다음 hop으로 전송이 제대로 안된 경우.
- Ethernet의 경우가 한 hop사이니, 회복 시간이 더 짧아서 좋다.
- 하지만, Ethernet의 경우, `오로지 충돌이 일어난 경우`만 재전송이다. `Collision detection이 안 된 경우, 재전송은 없다`.
    - 만약, collision detection에 실패한 경우, 문제가 생긴다. 하지만, Ethernet은 `유선 네트워크`라 `바깥의 noise가 개입할 여지가 거의 없어서` 문제 없다.
    - collision이 없으면 사실상 `도달 확정`이니, gateway router에서 ACK을 보내주지는 않는다.


#### 실패한 detection
- 근데, 정말 만에 하나 collision detection에 실패한 경우에는?
    1. 한 bus 내에 A, B, C, D, E가 있다(순서대로).
    2. A가 정보를 전송하고 있다.
    3. 하필이면 A의 신호가 도달하기 전에 E가 carrier sense를 수행했다. E는 지금 data를 전송 시작했다.
    4. 그 직후, E는 A의 신호를 받고 collision detection을 하게 된다. 이제 A, E 둘 다에서 보낸 신호는 쓸 수 없게 된다.
    5. A에 E가 기존에 보냈던 신호가 도달하기 전에 A의 전송이 끝났다. 그런 경우, A는 collision이 났으나 이를 detection할 수는 없다.
- A가 `조금만 더 길게 끌었으면` 제대로 `collision detection`을 할 수 있었는데... 원흉은 `propagation delay지만, 이것을 없앨 수는 없다`. 그렇다면 A가 할말이 없어도 조금 더 끌어도 좋지 않을까?
    - `Minimun Frame Size(64 byte)`를 만들어 최소한 이정도 이야기하도록 해서 detection이 가능하게 한다.
    - 만약 A가 원래 1 byte만 이야기한다 해도, padding을 일부러 넣어 Minimum Frame Size를 준수하도록 한다.

## Address
- link layer의 frame에 들어가는 address는 `MAC address`다.
- 48 bits.
- `1A-2F-BB-32-10-2A` <- 이런 format이다.(16진수수 2개로 총 6자리. 8 * 6 bit)
- 앞의 24bit는 제조 회사, 뒤의 24bit는 해당 interface의 고유 번호.
- 현실 세계의 사람에게 이름, 주소, 주민번호가 있다면 네트워크는 host name, IP address, MAC address에 대응된다.
    - host name, IP address는 바꿀 수 있지만, MAC address는 `바꿀 수 없다`.
    - MAC address는 network interface card가 `공장에서 찍어 나올 때 할당되는 것이다`. 바꿀 수 없다.
- link layer는 `하나의 인터페이스에서 다른 인터페이스로 보내는 것`이니 이 MAC address를 사용한다.
    - 밖으로 나가는 frame에서 MAC address의 source destination을 변조할 수 있다. 내 진짜 MAC은 그대로지만, 이렇게 해킹을 할 수 있다(맥 스푸핑). 자세한건 보안 시간에.

### 통신을 해보자
- 나 자신의 source MAC address는 알고 있다. network 계층에서 source IP address는 알고 있다(NW 계층에서 DHCP protocol을 적용). `패킷의 Destination IP address`도 알고 있다(DNS). 하지만, 나는 `gateway router의 MAC address를 모른다`.
    - gateway router의 IP address는 일단 알고 있다(DHCP protocol). 이걸 이용해 MAC address를 찾아보자.

### ARP(Address Resolution Protocol) table
- IP 주소와 MAC 주소가 매핑된 테이블. host 자신이 갖고 있다.
- 이를 이용해 MAC address를 찾을 수 있다.
- TTL도 있다. `해당 정보의 유효한 시간`이다.
- 근데 이걸 어떻게 채웠을까?

## ARP protocol
- ARP table에 내가 찾고자 하는 주소가 없다. `그러면 ARP request frame을 LAN 전체에 broadcast`한다. source는 나고, destination은 `all 1(broadcast)`. message 안에는 `gateway IP가 적혀 있다`.
- 수신 측은 data를 보고, `내 IP면 응답을 해준다`. `내 MAC 주소`를 여기 실어서 보내준다.

### 주의)
- 예를 들어 A -> gatewayrouter -> R1이라 하자.
- 이 때, A -> GWR에서의 destination MAC address와 GWR -> R1에서의 source MAC address는 `다르다`. 다른 인터페이스이기 때문이다.
- router의 매 hop을 거쳐갈 때마다 `IP packet은 변하지 않는다`. frame의 헤더만 바뀐다.
    - 이 때, `forwarding table과 ARP table을 참조`하며 간다.