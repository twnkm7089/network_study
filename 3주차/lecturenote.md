# 애플리케이션 계층 1

## 소켓 프로그래밍

### 소켓이란?
    - Application layer의 사용자 입장에서는 통신은 클라이언트 프로세스랑 서버 프로세스 사이의 통신이다. 이 때, 사용자는 제공되는 인터페이스를 사용해야 한다. 이러한 것은 OS가 제공해주는 API의 일종이고, 이를 `소켓`이라고 한다. 그리고, OS가 제공해 주니, OS에서 제공해 주는 것만 사용할 수 있다.
    - Application layer의 밑에는 Transport layer가 있고, 우리는 이곳에서 제공하는 소켓을 사용해야 한다. 그런데, OS는 TCP와 UDP만 제공하니 우리는 이 둘 중 하나를 위한 소켓을 만들어 사용해야 한다.

### 소켓에는 TCP, UDP 소켓 두 종류가 있다.
    - TCP 소켓은 소켓 stream이라 불리고, UDP 소켓은 소켓 datagram이라 불린다.


## 소켓 functions(TCP)
- TCP case
    - TCP server
    1. socket().서버가 소켓을 연다.
    2. bind(). 방금 생성한 소켓을 특정 포트에 바인드한다.
    3. listen(). 서버를 위한 것이니 이 소켓은 듣는 용도다.
    4. accept(). 나는 클라이언트로부터 요청 받을 준비가 되었으니 들어와라.

    - 이 이후, 서버는 클라이언트로부터 커넥션 들어올 때 까지 blocking 상태가 된다. 즉, 멈춰 있다. 후에 커넥션 들어오면 작동한다.

    - TCP client
    5. socket()으로 소켓 생성
    6. connect()로 TCP 연결 생성. 이 과정 끝나면 둘 사이의 단단한 연결고리 형성됨.
    7. 이 이후로는 단순하다. 연결이 맺어졌으니 이 소켓에 메시지를 write()하면 전 세계 수많은 소켓중 현재 이어진 상대 소켓에 메시지가 보내지고 상대방이 read()한다.

    8. 다 끝나면 close()한다.

### 소켓 생성과 setup
- include file<sys/socket.h>
- create로 생성. 인자는 3개. 진짜 중요한건 2번째 인자로 TCP, UDP중 어느 소켓 생성할 것인지를 결정한다. 생성이 완료되면 소켓 관련 자료구조가 만들어지고, 반환값으로 file descriptor나 -1이 나온다. file descriptor가 이 소켓을 지정한다.
- bind. 이 소켓을 특정 address에 bind한다.
- listen. 방금 생성한 이 소켓을 listen용도로 하고, 동시에 들어오는 것을 backlog개까지는 처리하겠다.
- accept. 나는 다 했으니 이제 클라이언트의 연결을 기다리겠다.
    - 새 커넥션이 생겼을 때, accept의 cliaddr에 클라이언트의 IP와 port번호가 저장된다.

- connect. 당연히 특정 소켓과 연결해야 하니 해당 소켓의 정보를 인자로 넘겨야 한다.
    - bind에서는 왜 소켓번호를 안쓸까? 남는데 아무데나 넣으면 되므로.


### 샘플 코드
- 이 서버의 포트는 3490, 최대 10개 처리.
- 이 소켓은 SOCK_STREAM, 즉 TCP 연결. sockfd에는 이제 소켓 정보가 지정된다.
- bind로 이 소켓을 특정 address에 bind한다. sockaddr이라는 구조체를 쓰는데, 구조를 보면 내 정보를 적어준다.
- 이제 listen으로 소켓 받을 준비 됨 알리고, block후 accept로 받을 준비 됨을 알린다. 후, connection 요청 들어오면 두번째 인자로(&their_addr) 클라이언트 소켓 주소 저장되고 커넥트 완료.

- 클라이언트는 소켓 생성한 후, 바로 connect한다. 두번째 인자로 이 경우는 서버의 주소가 들어간다.

- 이후는 read & write다.


## 소켓(UDP)
- 훨씬 단순. 소켓 생성하고 connection같은거 없이 바로 전송.
- 안쓸거라 따로 설명은 안함.


## close
- 데이터 교환이 다 끝나면 close로 연결 끝내서 다른 프로세서가 이 소켓 쓸 수 있도록 만들어준다.

### release of ports
- ctrl+C 눌렀을 때와 같은 경우, 프로세스는 죽지만, 포트 정보가 그대로 남아있는 경우가 있다. 이러면 이 포트는 계속 접근이 불가능해지니 문제다.
- 이를 방지하고자, 특정 코드를 삽입해 System call 같은 방식을 통해 이러한 입력이 들어오면 자동으로 포트를 release하도록 할 수 있다.


## project 1 설명
- 웹서버 구현
- C로 구현, 우리가 봤던 HTTP request, HTTP response를 보게 된다.
- 클라이언트 구현하는거 아니다.
- 웹 브라우저에서 뭘 치면 HTTP request를 보낸다. 근데, 이 이전에 이미 TCP 소켓을 열어 connection을 해 놓았을 것이고, 이 소켓을 통해 정보를 주고 받고 있었을 것이다.
- 이제 서버에서 해당 소켓을 이용해 HTTP request를 read()하여 화면에 띄우면 된다.
- 그 다음에는 요청하는 파일을 parsing해서 원하는 파일을 HTTP에 실어서 HTTP response에 실어 보내는 것이다.
- 컴퓨터 한대에서 포트번호만 다르게 하면 한 컴퓨터에서도 실험 가능하다. localhost:7777/index.html같은 방식.

- 아무것도 없이 온 요청이 있으면 index.html을 돌려준다. 근데, 여기 hypertext reference로 파일들이 있었으면 클라이언트측에서 웹 브라우저 파싱하던 과정에서 이를 파악해 다시 요청한다.


## 전송 계층
- TCP든 UDP에서든 전송 계층에서는 기본적으로 제공하는 공통 기능인 multiplexing, demultiplexing에 대해 알아보자.

### MUX, DEMUX
- 컴퓨터에는 여러 네트워크 프로세스가 있고, 각자의 포트가 있을 것이다.
- transport layer에서는 어디서 오든 하나의 segment로 만들어 아래 계층으로 내려준다.
- 어느 소켓에서 온 데이터든 뭉쳐서 하나로 만들어 주는게 multiplexing. 이제 나중에 데이터가 도착해 segment를 다시 나누어 메시지를 보고 적절한 포트로 보내주는게 demultiplexing이다.
- 즉 multiplexing은 sender, demultiplexing은 receiver 측에서 하는 것이다.

### DEMUX는 어떻게?
- 근데 demultiplexing 할 때 올바른 소켓을 어떻게 판단?
    - segment에 있는 header를 보고서 판단.
    - segment는 data와 header로 이루어진다. 헤더 내의 필드 중 source port number, destination port number가 있다. 이를 바탕으로 demultiplexing을 한다.
    - 참고로, 데이터 부분이 헤더보다 엄청나게 크다.

- UDP의 경우
    1. Process3와 Process1(서버)가 소통 원하고, Process4도 Process1과 소통 원한다.
    2. 통신을 위해 3, 4 둘다 UDP 소켓을 연다. 1도 UDP 소켓을 연다.
    3. 메시지가 소켓을 통해 전송된다. segment에는 source port와 destination port가 나온다. 참고로, 다른 컴퓨터로 가기 위한 IP 주소는 아래의 Network layer에서 생성된 segment의 header에 붙는다.
    4. UDP를 사용할 경우 demultiplexing 시 `destination IP와 destination port만을 이용해` 어느 소켓으로 올릴지 결정한다. 그러면 3, 4 둘 다 destination IP, port는 같다. 그래서 둘은 같은 소켓으로 간다.

- TCP의 경우
    - TCP의 경우 demultiplexing 시 destination IP, port 뿐만 아니라 `source IP, port도 사용`한다. 그래서 `이 넷 중 하나만 다르면 다른 소켓`으로 간다. 그래서 위의 상황의 4에서 두 segment는 다른 소켓으로 간다.
    - 이게 `TCP의 connection-oriented다. 한 소켓에 들어올 수 있는 것은 전 세계에서 이 소켓과 연결된 단 하나의 소켓에서만 가능하다.` 반면, UDP는 destination만 같으면 아무나 다 올 수 있으니 connection이라 할 수 없다.
    - 실제 구현은 각 프로세스가 있고, 거기에 있는 각 thread별로 socket을 만든다. 즉, 각 사용자별로 그 사용자만을 위한 socket이 만들어지고, 연결된다고 생각하면 된다. 이러다보니 `자원을 많이 소모`한다.

- 질문
    - broadcast의 경우 destination은 안 정해져 있다. 현재 인터넷에서는 broadcast를 제공해 주고 있지 않다. 카톡방에서 4명에게 broadcast하는 것처럼 보이는 건 사실 네명에게 TCP connection이 있는 것.

## UDP
- UDP가 제공해주는건 unreliable delivery, unordered delivery
- 언뜻 보면 아무것도 제공 안해주는 것 같으나, transport layer가 제공해주는 기본적인 것은 제공해준다.

### UDP의 setment header
- 필드가 source port number, destination port number, length, checksum의 4개밖에 안된다. 단순하다.
- source port number, destination port number는 각각 16비트다. 즉, port number는 0 ~ (2^16-1 == 약 60000개)까지만 존재한다. 이걸 가지고 multiplexing, demultiplexing을 한다.
- length는 데이터 길이다.
- checksum은 담긴 데이터의 에러를 파악하는 것이다. 전송 도중 에러가 생겼는지 아닌지를 이를 통해 판단할 수 있고, receiver는 이걸 확인해 에러가 있으면 application layer로 안 올리고 그냥 drop을 한다.
- 즉, UDP는 `multiplexing, demultiplexing, error checking`의 역할은 한다.

- TCP는 이것보다 많겠죠?

- UDP, TCP, IP header는 중요하니 잘 알아두자. 이 정보가 프로토콜의 원리를 나타낸다.