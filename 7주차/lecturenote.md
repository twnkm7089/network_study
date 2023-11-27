# 전송 계층 3

- flow control, connection management

## flow control
- A와 B 사이의 TCP 연결이 되어 있다. 각자 자기 자신이 보내는 데이터의 속도를 조절 가능하다. 받는 사람의 처리 속도에 맞추어 보내야 한다.
- 둘에게는 각자의 sender buffer, receiver buffer가 존재한다.
    - receiver buffer에서는 오는 데이터를 쌓아서 application layer로 올려준다. socket programming할 때, read()로 데이터 읽어왔죠? 그게 바로 여기서 읽어들이는 것이다.
- 만약 B의 receiver buffer에 남는 공간이 조금 있다고 하자, 만약 A의 sender buffer에서 너무 많이 쏘면? 의미가 없다. 그래서 receiver buffer의 빈 공간에 맞추어서 보내야 한다. 그래서 flow control은 `receiver-driven`하다. receiver가 결정하기 때문이다.
    - 그렇다면 얼마나 비었는지를 어떻게 알려줄 수 있을까? `TCP header`에 있는 `receive window`라는 필드에 넣어서 보내준다. 수신 측에서 자신의 receiver buffer에 남는 공간을 계속 report해주며 feedback을 날려준다.

### 질문
- 그럼 flow control은 속도를 조절하는 것인가, 보내는 양을 조절하는 것인가?
    - 속도를 어떻게 표시하죠? bps. 1초에 몇 bit씩 보내는가? 단위시간에 보내는 양이 많으면 속도도 많다고 할 수 있다. 즉, 따지고 보면 보내는 양을 조절하는 건데, 이게 곧 보내는 속도를 조절하는 것이다.

## flow control 추가
- 남는 양도 있지만, application layer에서 읽어들이는 속도도 결국 빈 공간의 크기를 결정하니 읽는 속도를 결정한다.
    - 만약, application layer에서 바빠서 안 읽어서 남는 공간이 0이 되면? 그러면, 이게 report 되어서 sender는 더이상 데이터를 보내주지 않는다.
    - 근데, 만약 이 상황에서 B가 데이터를 읽어들여 공간이 생겼다. 하지만, 이 정보는 report되지 않았으니 A는 아직 모른다.
    - 이런 corner case에 대해서 TCP는 공간이 하나도 없을 때, `주기적으로 A에서는 의미 없는 segment(예를 들면 data공간이 빈 segment)를 보낸다`. 아무 의미 없는 segment를 보내어서 상대방에 빈 공간이 생겼는지 feedback을 받는다.

## connection management
- 양쪽에서 data를 주고 받기 위해서는 우선 양쪽에 buffer 2개가 각각 있어야 하고, 내 sequence number도 알아야 하고(sender buffer쪽 문제), 상대방의 sequence number도 tracking해야 한다(receiver buffer쪽 문제).
    - 이러한 구조체를 처음에 만들어 놓아야한다. 이것이 바로 `connection establishment`다. 커넥션을 만들어 놔야 한다.

### TCP 3-way handshake
- 우선 클라이언트가 서버에 TCP 커넥션을 열자고 요청해야 한다. 아직 데이터는 왔다갔다 하지 않는다(데이터는 연결 후에...).
1. 클라이언트가 서버에 TCP 연결을 열고 싶다고 요청한다(`SYN segment`). 데이터 부분은 아무것도 없고, header의 `SYN 필드`가 1로 setting되어 있다. 또, 나의 `sequence number를 알려줘` 상대방이 tracking할 수 있도록 해야 한다.
2. 서버는 '어 TCP 열자'라고 답변한다. 이게 `SYN ACK`이다. `SYN은 1`이고, ACK number는 클라이언트가 보내준 `sequence number + 1`. sequence number도 보내준다.
3. 클라이언트는 SYN ACK에 대한 `ACK`을 보낸다. `SYN은 0`이고, 이 때부터 `데이터 포함 가능`하다.

- 이렇게 3번에 걸쳐 진행되므로 3-way handshake라고 불린다. 이 결과로 TCP 연결이 이루어진다.
    - 근데, 그러면 이미 2번에서 완료된거 아닐까? 왜 3-way일까?
    - client입장에서야 2번에 연결된 것을 인식하지만, `server입장에서는 2번 상태에서는 feedback이 안 와서 제대로 형성된 것인지 모른다`. 3번째 ACK이 돌아올 때가 되어서야 server는 buffer를 형성한다. 즉, 3번이 끝나야 server, client 모두 연결을 인식할 수 있다.

#### 과거에 HTTP 연결할 때...
- HTTP request, response 나오기 이전에 TCP connection이 왔다갔다 하던 것을 기억하는가? 그게 바로 TCP 3-way handshake다.
    - 처음에 TCP SYN, 두번째 SYN ACK, 마지막에 HTTP request로 보내지는 것이 사실 SYN ACK에 대한 ACK에 정보를 실어 보낸 상황이다.

#### 이제 보낼 자료 다 보냈다... 이제 끊고 싶다.

### Connection close
1. client가 `FIN segment`를 보낸다. FIN이 1로 되어있다.
2. 이걸 받으면, 서버가 마지막으로 보낼 data를 클라이언트에 마저 다 보내고, 마지막에 FIN segment를 보낸다. 그리고 `server는 Close WAIT`를 한다(passive close).
3. client가 이 FIN까지 받았다. 하지만, 아직 닫지 말고 `timed wait`을 한다. 왜?
    - client는 FIN을 받고, ACK을 보낸다. 근데, 이 ACK이 소실되었다면? server는 timeout되었으므로 계속 다시 보낼 것이다. 이를 방지하기 위해 client는 잠깐 더 열어두었다가 `연결을 닫는다`.
    - timeout값은 고정이 아니라 항상 바뀌므로, client는 server의 timeout을 모른다. 그냥 좀 더 기다릴 뿐이다.
4. server가 ACK을 받고 `closed`된다.

## 간략히 congestion control의 의미와 TCP는 어떻게 접근하는지 알아보자.
- A, B 둘 사이에는 network가 있다. A의 sender는 B의 receiver와(flow control) 네트워크 사정(congestion control)에 맞추어 데이터를 보내 주어야 한다.
- Network와 receiver의 능력치 한계 중 어떤 쪽에 더 맞춰 주어야 할까? 
    - receiver가 10만큼 처리 가능한데 network가 20만큼 처리 가능하면 -> receiver에.
    - receiver가 10만큼 처리 가능한데 network가 5만큼 처리 가능하면 -> network에.
    - 둘 중 `상태 나쁜 쪽`에 맞추어야 한다.
<br>

- 근데, receiver의 상태는 알 수 있지만, `network의 상태는...? 확실치 않다`. 이를 알기 위해 `congestion control`을 하여 알아낸다.
<br>

- A와 B 사이에 internet network가 있다. 이 network는 public이다. 주인이 없다. 그래서, network에는 A와 B뿐만 아니라 모두가 달라붙어 있다. 모든 사람들은 `많이 쏟아 붓기를 원한다`. 그러면 `막힌다`.
    - 막힌다의 의미는? 막힐 때, 더 많이 부으면 악화된다. `하지만`, TCP의 입장에서는 막히면 잘 안가고, 패킷이 유실되거나 느리니 재전송한다. 실제 데이터 양보다 `더 많이 보낸다`. 그래서 더 막힌다.
    - 그렇다면, TCP는 어떻게 해야 할까? 네트워크가 막히지 않게 해야 한다. 즉, 데이터를 `마구잡이로 많이 보내면 안된다`. 네트워크가 막힐 것 같으면 `모두가 data rate를 줄여야 한다`. TCP는 네트워크가 막힐 것 같으면 data rate을 줄이고, 반대로 안 막힐 것 같으면 data rate을 늘린다. 어떻게? `window size를 조절`하면서...

- 근데 이걸 어떻게 알건데? 크게 2가지 접근 방식이 있다.
    - Network-assisted congestion control : 네트워크의 router들이 자신의 상황을 알려줘서 TCP가 조절할 수 있도록 한다. `현재 구현되어 있지 않은 이야기`다. 그냥 꿈일 뿐...
    - End-end congestion control : `실제 구현된 방식`. `양 끝에서` 내부 상황을 알아서 `유추`해서 보내준다. 이 방법은? segment를 보냈는데 ACK이 안 오거나 느리게 오면 문제 생긴 것. 이러한 방식으로 유추할 뿐이다.
