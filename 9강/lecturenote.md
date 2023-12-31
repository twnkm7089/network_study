# 네트워크 계층 1

## 지난시간을 훑어보며
- DNS는 왜 UDP를 사용하는가?
    - 속도가 빠르고(DNS는 신뢰성 보다는 속도, 전송 패킷 사이즈가 작아 UDP로도 충분하고, 오버헤드 없어 속도 빠름(패킷 오버헤드, 헤더에 추가되는 유지보수 위한 정보, UDP에서는 최소화된다.)).
    - 연결 상태 유지 불필요(TCP보다 많은 클라이언트 수용 가능).

- 잠깐? 최대 패킷 사이즈?
    - UDP 페이로드는 RFC 768(1980.07)에 의해 16bit의 length 필드를 가지므로, 이론적으로 65536(2^16)bytes - 8bytes(UDP헤더 길이) = 65,527bytes까지 가능하다. 하지만, total length는 IP헤더를 포함한 데이터그램의 전체 길이로 정의되므로, RFC 791(1981.09)에서 IP헤더 최소 길이 20byte를 제외한 65,507bytes로 바뀌었고, 사실상 장비의 문제 등으로 `최대 처리 가능 데이터그램은 576bytes`다. 이는 RFC 815(1982.07)에 fragmented datagram 재조립 위한 알고리즘을 정의하며 576bytes 이상의 길이는 선택 사항으로 남긴 것에 드러난다. (출처 : https://scrapsquare.com/notes/udp-length)
    - TCP의 경우 기본 최대 세그먼트 크기(MSS)는 `1460바이트`, 최대 전송 단위(MTU)는 일반적으로 이더넷 기준인 1500바이트.

- TCP 문제
    - ACK번호 구할 때, sequence number와 data size 둘 다 고려해야 되는 점 주의
    - window size에 따라 달라지는 나가는 segment의 개수
    - 'fast transmit'
    - MSS(Maximum Segment Size)단위 기억(at congestion control)


## Network layer introduction

### 계층에 대한 복습
- 네트워크는 복잡하니 잘 관리하려고 계층화 했다.
- 상위 계층일수록 개념적, 하위 계층일수록 뭔가 구체적.
- 쌓은 형태라 stack이라고도 함.

### 이제 네트워크 레이어를 보자.
- segment는 그래서 구체적으로 어떤 식으로 갈까? 지금까지는 그냥 간다고만 했다. 이제부터 봐보자.
- 그 배송을 network layer에서 해주고, 그 프로토콜이 `IP`다.
- `라우터`가 이 보내주는 것을 담당한다. 따라서, `라우터는 네트워크 계층까지 존재`한다.

### 네트워크 계층의 일
- source부터 destination까지 `어떠한 경로로 보내줄지` 결정.
- `라우터`들이 어떠한 방식으로 개입해서 보내주는가?
- 라우터는 패킷을 받아서 network layer까지 끌어 올린 후, 패킷을 보고 어느 경로로 보내주어야 할 지 결정해 보내준다.

### 라우터의 역할
- 크게 다음 두가지다.
    - forwarding
    - routing

- 패킷에 들어온 것을 목적지의 방향으로 전달한다.
- 헤더를 보고 `거기 적힌 목적지`를 보고, `거기 보내주기 위한 링크가 어떤 곳`인지 판단한다.
- 각 라우터에는 `테이블`이 있다. 이 표는 `특정 목적지로 가기 위해서는 어디로, 몇 번 인터페이스`로 가야할 지가 저장된다. 이렇게 `패킷의 목적지를 보고 해당 링크로 보내는게 forwarding`이고, 이 테이블을 `forwarding table`이라고 한다.
- `forwarding은 단순`하지만, `forwarding table이 미리 만들어져 있어야` 한다. 이걸 누가 만들까?
    - 이걸 자동적으로 만들어주는 것이 `routing`이라는 작업. 이 때 쓰이는 알고리즘이 `routing algorithm`.
    - 모든 목적지가 저장되어 있으려면 너무 table이 커지고 비효율적이다.
    - 그래서 `주소 범위로 forwarding table이 관리`되어 있다. '주소 x번부터 y번은 n번 인터페이스로 나가라.' 같은 식이다.
        - 주소에 `*`은 여기에 아무거나 들어와도 된다는 의미이다.
        - 여러 주소에 맞는 경우, `가장 구체적으로 맞는 곳`으로 간다. 가장 길게 맞는 엔트리로 보내니 이를 `longest prefix matching`이라고 한다.