# 무선이동네트워크 2

## Wi-fi
- 2.4GHz, unlicensed band(라이선스 필요 없이 사용 가능).
    - 실제로는 이 대역이 `11개의 서브채널`로 나뉘고 이 중 한 채널을 선택하는 방식. 선택은 `AP`가 한다.
    - `서브채널이 다르면 간섭은 일어나지 않는다`. 따라서, 근처에 있는 AP가 같은 채널이 아니면 이들에 연결된 host끼리는 서로 간섭이 나지 않는다(`collision domain 분리`).
    - 만약 서브 채널이 같아 간섭이 일어나면? CSMA/CA, RTS, CTS
    - 동일한 서브채널을 쓰는 host끼리는 서로 경쟁. host가 많을수록 충돌도 많이 발생, 속도 저하.

## CSMA/CA
- link layer ACK의 존재.
- RTS, CTS를 이용. 충돌은 결국 피할 수 없는데, 크기가 작은  RTS, CTS를 이용하면 그나마 문제 시 회복 빠르다.

## frame addressing
- header, data, tail
- address field는 각 6bytes, 4개 있으나, 3개만 사용. 모두 MAC address 사용.
- address2는 이 무선 프레임을 전송하는 인터페이스의 맥 어드레스, address1은 수신측 인터페이스(AP), address3는 라우터 어드레스.

### AP
- 양쪽 프로토콜이 다르다. 한쪽은 wi-fi, 한쪽은 ethernet
- wi-fi쪽의 인터페이스는 `host 입장에서는 보인다`. 하지만, router입장에서는 `AP의 인터페이스를 인식 못한다`.
    - 즉, AP에서 변형된 이더넷 프레임에서, `source address는 host의 address(AP의 어드레스 아님.)`. wi-fi frame에서 host, AP, router를 쓰듯, ethernet은 host, router만 쓴다.
    - AP는 `link layer device`다.
    - wi-fi frame의 data로 들어가 있는 IP packet을 보면, destination IP가 저장되어 있다. 이걸 router로 고치는 것이다.
- 수신 시, 구글에서의 응답이 h1으로 향한다 가정하자. ethernet에서 `source는 router, destination은 h1`
    - wi-fi frame에서 `address2`는 무선 프레임을 전송하는 `AP`다. address1은 무선 프레임을 받을 `h1`, address3는 무선 프레임이 전송 시작된 곳(`router`).
- AP에 연결된 host는 많다. 여러 개의 host가 AP 한 곳으로만 이동한다. 즉, `multiplexing`이 일어나고 있다.
    - 그러니 다음에 어디 가야 할지 필드 3개를 표시한 것이다.
- address1, address3를 적어주자. 
    - address1(AP MAC 주소는)? AP가 주기적으로 `becon frame`을 보내준다. 거기 AP의 MAC address가 담겨 있다.
    - address3(router MAC 주소)>? h1 노트북의 뚜껑 열어서 becon message 받은 후, `나 자신의 IP 주소`를 알아야 한다(MAC은 알고 있다). `DHCP`를 이용해 이를 찾는다.
        - DHCP request를 만들기 위해 frame을 만든다. address1, 2(AP, h1)는 안다. 하지만, address3는 broadcast한다. 이더넷 프레임으로 전환 후 퍼진다.
        - broadcast로 DHCP offer 도착. 그 후는 우리가 아는 DHCP 상황.
        - 나(h1)의 IP주소, subnet mask, gateway router의 IP, local name server의 IP를 얻었다. 여기서 ARP table을 이용해 `gateway router의 MAC address`를 얻는다. 없으면 `ARP query`를 만들어 `broadcasting`. 돌아 올 때는 정보를 아니 `unicast`. 그래서 ARP table은 DHCP 과정에서 자연스럽게 만들어진다.

### 질문
- DHCP 갔다가 올 때(offer), AP에서 frame은 어떤 식으로 바뀌는가? address2는 AP, address1은 broadcast가 된다. address3는 그냥 router 주소다.
    - broadcast 된다. 정보는 IP packet을 까봐야 나온다. 본인 정보면(열려 있는 port번호) 받는다.
    - 여럿이 DHCP 중이어서 해당 포트가 여러 데에서 열려있었다면? application layer에서 처리.

### 순수한 AP vs 가정용 공유기
- 순수한 AP는 link layer만 존재. 회사에서 사용.
- 가정용 공유기는 application layer까지 존재하는 작은 컴퓨터. 그래서 DHCP server, local name server 등의 역할도 한다.

### frame 그외
- frame control(2 bytes) : 여러 부분으로 나뉜다.
    - type : RTS, CTS, ACK, data 중 하나. data가 길고, header는 overhead니 최소화 위해 2bit 사용.
- duration(2 bytes) : RTS, CTS 시 앞으로 얼마나 쓸 지 시간 설정.

## Mobility 상황에서(같은 subnet 내부)
- 303호에서 스마트폰으로 영화를 보다 304호로 이동한다. AP는 바뀐다. 연결은 유지되는가?

### 연결(Connection)
- 여기서 connection은 TCP connection이다.
- client와 server 사이의 TCP connection이다. 즉, socket이랑 socket 사이의 연결, 전 세계 유일의 연결.
    - 이 TCP connection은 뭘로 인덱싱될까? 포트번호? 모두가 이 포트번호로 접속하는데?
    - `source IP, port번호, destination IP, port번호` 4가지를 `모두 사용`하여 indexing한다.
- 만약 이 넷 중 하나가 `변하면 끝난다`. 이동을 하면? destination은 그대로니 source(즉, 나 자신)의 IP와 port 번호의 변화를 보면 된다.
- 같은 subnet 내이므로 source IP, port번호는 변하지 않는다. 고로, `끊기지 않는다`.
- 다만, AP(혹은 Base station)가 바뀌었으니 routing이 수정된다. AP 이전의 경우(이더넷 프레임) `destination은 h1, source는 router`이다.
    - forwarding table을 본다. `도움이 안된다`. 같은 subnet에 있으므로 h1으로 가려면 h1으로 가세요 같은 방식이다. 주소 설정해 내려 보낸다.
    - `switch`에 도달했다. `switch table`을 참조해 AP를 결정해 보내준다. 원래 mapping된대로 가면 AP1으로 그냥 내려간다. 하지만, `이미 AP2로 host가 이동`했다. 어떻게 `바꾸어야 하나`?
    - 결국 switch table은 self learning을 통해 형성된다. `h1이 AP1 -> AP2로 이동할 때`, `더미 메시지를 보내면` 알아서 table이 `갱신`된다.
<br>

- 만약 `다른 네트워크`면? `연결이 끊긴다`. IP 주소가 바뀌기 때문이다.
    - 근데, 3G, LTE 쓸 때는 base station은 계속 바뀌는데 연결이 끊기지 않는다. 어째서?
    - 같은 네트워크에 속한 base station들이 전국적으로 깔려 있다. 즉, `네트워크 범위가 전국`이라 끊기지 않는다.