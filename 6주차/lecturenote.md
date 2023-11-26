# 6강 전송계층 2

## 정리
- TCP는
    - `point-to-point` 통신을 권장한다. : 딱 한 쌍의 연결을 권장. 즉, `소켓 한 쌍 끼리`의 통신을 책임진다(프로세스에 소켓 여러개 있어도 마찬가지 하나씩만).
    - `reliable, in-order byte` : 신뢰가능, 순서대로
    - `pipelined` : 한번에 여러 패킷 보낸다.
    - `full duplex data(양방향 데이터 통신)` : 지금까지는 sender, receiver로 봤지만, 사실은 모두가 `sender인 동시에 receiver`이다.
    - `send & receive buffers` : sender의 buffer와 receiver의 buffer가 window의 크기만큼 필요하다. pipelined한 통신을 진행하므로.
        - 참고로, 두 소켓 다 sender이자 receiver이므로 둘 다 각각에 대응되는 sender buffer와 receiver buffer를 가지고 있다.
    - `connection-oriented` : 연결을 지향한다.
    - `flow control` : receiver의 소화 능력에 맞게 데이터를 보내준다.
    - `congestion control` : 내부 네트워크가 받아 줄 수 있는 양만큼만 보내주게 조절한다.

## TCP segment structure
- TCP segment의 구조는?
- 네트워크는 복잡해서 계층화, 추상화 되어있다.
    - Application layer의 데이터 전송 단위는 `message`다. 실제 편지지에 적힌 내용과 같은 느낌이다.
    - Transport layer의 전송 단위는 `segment`다. DATA 영역에 message가 들어가고, HEADER에 부가적인 내용이 들어간다.
    - NETWORK LAYER에서는 전송 단위는 `packet`이 된다. DATA에 segment가 들어가고, 해당 계층의 HEADER에 부가적 내용이 들어간다.
    - LINK LAYER에서는 전송 단위는 `frame`이고, 위의 packet은 DATA에 들어가고, HEADER에 부가적 내용이 들어간다.
- 우리는 지금, TCP를 다루고 있으니, segment에 대해 알아보자. 특히 header에 대해서 말이다. data는 위에서 내려온 message이므로.

### TCP header
- 구성 요소
    - `source, destination의 포트번호` : 각각 16bit, 즉, 이론상으로 약 60000개 정도의 포트 번호가 가능하다.
        - source는 보낸 측, destination은 받는 측.
    - `sequence number`(32bit)
    - `acknowledgement number`(32bit)
    - `checksum`(16bit) : 오류 체크.
    - `receive window` : 내 포트에 있는 receive buffer에 남는 빈공간이 얼마나 있는지 피드백 주어야 상대방에 조절한다. 그걸 위해 있다. 16bit.
    - 그 외 이것저것 있다.

### TCP의 동작
- 우리는 호스트 A와 호스트 A의 내용을 그대로 재전송하는 echo server인 호스트 B의 통신을 예로 들어보자.
- 'C'라는 정보를 보내자. TCP의 data에는 'C'가 들어가고, header에 sequence number 42와 ACK number 79가 있다.
    - sequence number는 무엇일까? 우리가 100byte짜리 정보를 보낸다고 가정하자. 너무 기니 message를 10등분해서 보낸다고 가정할 때, 0~9번째 byte를 보낸다면 sequence number는 0, 10~19번째 byte를 보낸다면 sequence number는 10이 된다. 즉, sequence number란, `지금 보내는 것이 message의 몇 번째 byte인지`를 나타낸다.
    - ACK은 cumulative ACK이다. 하지만 Go-Back-N에서와는 다르다. TCP에서 ACK 10은 10번까지 잘 받았다는 의미가 아니라 `'9번까지 잘 받았고, 10번 내놓아라.'`라는 의미이다.
- 이를 통해 위의 정보를 해석하자, 'C'라는 것은 send buffer에서 42번째 bit였고, 나는 지금까지 78번째 bit을 잘 받았으니, 79번째를 내놓으라는 의미이다.
<br>
- 이제 B는 정보를 받았다.
    - receiver buffer는 42번 byte을 잘 받았고, 이제 43번을 받아야 하니, ACK 43을 segment header에 넣는다.
    - sender buffer는 79번 byte를 달라는 ACK을 받았으니, sequence number 79에 해당하는 byte를 보낸다. echo server이니 data에는 'C'가 들어간다.

#### 조금 더 응용해보자.
- 예를 들어, 원래 정보는 "COMPUTER"였다고 치자.
- Side A
    - sequence number는 제일 앞 byte의 번호다. 42였다 치자.
    - ACK은 79번이다.
    - 8byte가 갔다.
- Side B
    - B는 42~49번 byte까지 잘 받았다. 이제 이를 상위로 올리고, 다음에는 50번째 byte가 필요하다. `ACK 50`을 날린다.
    - A는 79번 byte가 필요하니, B는 sequence number는 79로 해서 A로 보낸다.
- 다시 말해,
    - sequence number를 결정하는 것은 위의 application층에서 도착한 message다. 여기서 몇 번째 byte였는지가 중요하다. A의 sender buffer는 B의 receive buffer와 연동되어서 관리된다.
    - 한편, A의 receive buffer에서 관리되는 번호는 B의 sender buffer를 트래킹 한다.
    - 즉, 한쪽의 receive buffer는 상대의 sender buffer의 번호를 트래킹한다.
    - sequence number는 sender buffer를 따라가고, ACK number는 상대의 sender buffer, 즉, 자기자신의 receive buffer의 영향을 받는다.

#### 질문 타임
- echo server B라고 해도 자기가 하고 싶은 말이 있을 것이다. A의 말에 대한 echo인지, B가 추가한 새로운 메시지인지는 어떻게 구분하지?
    - TCP는 이걸 할 수 없다. 이건 message의 내용이다. TCP는 이것을 관리할 수 없다.
    - Application layer에도 header가 존재한다. 즉, application layer의 header를 이용해서 구분해 주어야 한다. 당연히 이 application header도 segment의 data에 message의 일부로서 들어간다.
- 과거에는 누군가는 sender, 누군가는 receiver였다. 근데, 이제는 둘 다 sender이자 receiver이다. 그러면 ACK을 보낼 때 내 정보를 슬쩍 실어 보낼 수 있지 않을까? 그렇다면 ACK을 바로 보낼까, 아니면 내 데이터가 나올 때까지 기다렸다가 한꺼번에 보낼까?
    - 여기서 바로 대답할 질문은 아니고, 나중에 볼 문제. 우선은 '항상 보내고자 하는 데이터가 있다'고 가정. 우리는 이 데이터 보낼 때 상대방 ACK도 같이 보낼 것이다.
    - 간단히 답을 주면 `timing`이라는 것이 있어서 data가 들어오면 `우선은 잠깐 기다리고 ACK`을 보낸다. 이 이유는 `보낼 data가 생길지 몰라서 기다리는 것` 뿐만 아니라, pipelining을 통해 한꺼번에 여러 byte가 들어올 가능성이 있고, cumulative ACK을 사용하기 때문에 올 때 마다 ACK을 줄 필요 없이, 여러 byte를 기다렸다가 한번에 ACK을 보내는 것이 들어올 때마다 보내는 것보다 훨씬 경제적이다.

### 우리는 reliable transfer를 한다.
- 고로, 이를 위해 timer를 사용한다. timer value는 얼마만큼으로 setting할 것인가?
    - 작게하면 recovery는 빠르지만 네트워크에 쓸데없는 overhead를 주고, 크면 그 반대다.
- 그러면 우리는 유실을 확실히 가정하면서 timeout value는 최대한 작게 해야 한다.
<br>
- RTT(Round Trip Time)이 주어진다면, RTT보다 크면 유실, 작으면 아니다라고 생각할 수 있다. 그래서 우선은 timeout value를 RTT로 설정해보자.
    - A, B는 고정되어 있다. RTT값도 모든 segment에 대해 고정되어 있을까? 아니다. 우선, 각 segment들이 지나가는 경로가 다르고, 경로가 같다고 쳐도, queueing delay가 그때그때 다르게 발생할 수 있기 때문이다. 그래서 RTT값은 측정할 때마다 천차만별이다.
    - 이를 대표할 수 있는 RTT는 무엇일까? `EstimatedRTT`라는 보정한 RTT를 이용한다. 참고로, ppt에 있는 SampleRTT는 실제 측정한 RTT값이다.
<br>
- 그러나 이러한 보정한 RTT값을 써도 문제가 생긴다. 너무 작아서 premature timeout(유실 안 되었는데 timeout이 너무 빨리 온 것)이 일어난다. 좀 더 유실이 확실한 상황에 해야한다.
- 이를 위해 실제로 사용하는 것은 `estimatedRTT에 margin으로 deviation의 4배`를 붙여 `실제 TimeoutInterval`을 설정한다.

## TCP의 reliable data transfer
- pipelined 방식
- cumulative ACK
- TCP는 `timer 하나`를 쓴다. 마치 Go-Back-N같다. 하지만 다르다. timer가 터지면 다 재전송하는 것이 아니라, `해당 segment만 재전송`한다.

### 시나리오 1
- A와 B사이의 TCP connection이 연결되었다. A에서 segment가 나간다. seq:92, data-size:8byte. 즉, 92~99번 바이트가 전송되었다.
- 이제 B는 92~99 받아서 `ACK 100`을 보낸다.
- 근데, 오다가 유실되었다. timer가 expired되면 92~99 재전송하고, 다시 ACK 100.

### 시나리오 2
- 이번에는 92~99, 100~119을 보냈다.
- receiver는 ACK 100, ACK 120을 보냈다.
- 근데, ACK 100, ACK 120이 늦어서 92~99가 재전송된다. B는 보고 무시한다. 그리고 ACK 120을 보낸다.
- A가 다시 받으면 119까지 잘 받은 것을 알아챈다. 이제 싹 다 비우고, 120번부터 전송한다.

### 시나리오 3
- 위처럼 92~99, 100~119 보낸다.
- ACK 100, ACK 120날라간다.
- ACK 100이 유실되었다.
- 하지만, ACK 120을 본 A는 다 잘 받았다고 생각하고, 비우고 120번부터 보낸다.

### 근데, 우리는 RTT에 넉넉한 margin을 붙여놓았다.
- 그래서 이 timeout까지는 시간이 너무 길다. 좀 더 smart하게 할 수 없을까?
- data가 올 때마다 일일이 ACK 보낼 필요 없다. 그래서 TCP는 `delayed ACK`을 한다. 좀 기다렸다가 보낸다.
- 또, timer 터지기 전에 data 유실을 확인할 수도 있다. 실제 window size는 크므로, segment를 엄청 많이 보낸다. segment를 100개 보냈다 치자. 다 보내는데, 10번째 segment가 가다가 유실되었다고 치자. 그러면 `ACK 10이 반복적으로 도착`한다. receiver는 segment를 받으면 일단을 ACK을 무조건 보내고, cumulative ACK이기 때문이다. 이를 보고, `미리 10번 패킷의 유실을 알 수 있다`. TCP는 `duplicated ACK이 3번 도착하면 유실로 판단하고 미리 재전송하라고 권고한다(의무는 아님)`. 이를 `"fast retransmit"`이라고 부른다.
    - 주의) 일단 ACK 10을 받은 후, ACK 10이 `추가로 3번`올 때다. 즉,` ACK 10이 4번 왔을 때`다.