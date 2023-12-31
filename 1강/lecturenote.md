# Introduction

## 개요
- 컴퓨터 4대(노드 4개)로 연결된 작은 네트워크인 알파넷에서 이제는 엄청 커진 인터넷 네트워크.
- 우리는 이 인터넷 개념도에서 가장자리에 연결되어 있다. 심지어 웹 서버도 가장자리에 있다. 끄면 떨어져 나가고, 키면 붙으니까.
- 그럼 가운데에 붙는 애들은? 라우터. 메시지를 받아 목적지로 전달해주는 애들.

- 오늘의 목표) 인터넷의 구성 요소에 대해 알아보자.

## 구성 요소
- network edge : 우리의 랩탑, 데스크탑, 웹 서버 같은 애들이 붙어 있다.
- network core : router가 붙어 있다(동그라고 가운데 X 있는 그림)
- 이런 네트워크들을 이어주는 링크들 : 유, 무선의 다양한 링크들.

## 네트워크 엣지
- 서버와 클라이언트
    - `클라이언트`는 자기가 원할 때 링크에 연결해 `서버로부터 정보를 가져온다`.(우리 랩탑 같은 것들)
    - `서버`는 항시(24시간) 연결되어 있어서 `클라이언트로부터의 요청`을 항상 기다리는 것들(우리가 아는 웹 서버)
    - 네트워크 엣지에 있는 서버와 클라이언트가 서로 통신한다.

    - 그런데, 인터넷을 제공하는 통신 서비스를 통해 주고받겠죠?

## 데이터 통신 서비스
- Connection-oriented service
    - `TCP(Transmission Control Protocol)`가 바로 이 서비스를 제공하는 통신 방법. 이 서비스는 사용자에게 다음 `세가지를 제공`한다.
        - reliable(신뢰 가능한), in-order byte-stream data transfer : 메시지가 유실되지 않고 그대로 간다. 그리고 순서대로 간다.
        - flow control : sender가 receiver한테 데이터를 전송할 때, `sender의 전송 속도를 receiver의 처리 속도에 맞추어 알맞게 조절`해준다.
        - congestion control : 네트워크 상황에 맞추어 그 네트워크의 수용 가능한 수준에서 보내준다.
- Connectionless service
    - `UDP`가 대표적.
        - connectionless
        - unreliable data transfer
        - no flow control
        - no congestion control
    - 한마디로 아무것도 안해줌. 데이터 유실되든 섞이든 그냥 보내겠다 하면 이거 씀.

- 근데 TCP가 더 좋은듯한데 왜 UDP를 쓸까? 아무것도 안해주는데.
    - 속도가 빠르다. 물론, 막 보낸거라 의미가 없다. 하지만 유실되어도 상관없는 것(예 : 음성전화, 오디오 패킷 몇개 유실 되어도 문제 없음)에 쓰면 좋겠죠?
- 다만 대부분은 reliable한 TCP를 쓴다.

## 프로토콜
- 둘 사이의 메시지를 주고 받을 때의 약속.

## The network core(가운데에 있는 애들)
- 라우터가 있어서 데이터를 목적지까지 전달해준다.
- 라우터들이 옹기종기 엮여 있다.
- 궁금증? 라우터는 어떻게 데이터를 목적지까지 전달할까?
    - `circuit switching` : 출발지에서 목적지까지 미리 가는 길을 미리 예약해 놓고 특정 사용자만 이용하도록 한 것. 과거 유선 전화망. 물론 길을 독점하지는 않고, 길을 일부 할당해 그 사람만을 위해 할당(예를 들면 특정 길의 절반)
    - `packet-switching` : 패킷을 받아 그때그때 적절한 방향으로 전송. 그냥 들어온 순서대로 목적지로 전송, 전송. 인터넷이 이거 사용.

- 과연 둘 사이에 어떤 장단점이 있길래 인터넷은 packet switching을 선택했나?
    - N명의 users와 1Mbps link(초당 1Mbit 뿜는 케이블)가 있다 치자. 각 user는 100kb/s로 데이터를 보낸다.
    - circuit switching을 쓸 경우 최대 10명까지만 가능하다. 1Mbps까지만 되므로.
    - packet switching은 제약이 없다. 그냥 오는대로 보내주므로.
    - 물론, 너무 몰리면 문제가 생긴다. 11명 이상이 동시에 누르면 문제가 발생한다. 하지만, 그 확률은 낮다. 전화처럼 쉬지 않고 데이터를 전송하는 경우가 아니라 인터넷의 경우, 계속 클릭하지 않고 클릭한 후, 뭐 보고 클릭하므로 `데이터 전송이 이루어지는 시간보다 안 이루어지는 시간이 더 크다`. 그래서 circuit switching으로 할당하는건 비효율적이다.

## packet switching이라 생기는 문제들
- delay나 loss
- 라우터에서 패킷을 받아서 다음 라우터로 보내주기까지 얼마나 시간이 지연되는가?
    - 라우터가 패킷 받으면 괜찮은가 검사하고, 목적지 찾아서 보내주어야 한다. 이를 `processing delay`라 한다.
    - 그 후, 알맞은 곳으로 보내주어야 하는데, 유저가 몰려서 라우터에 들어오는 패킷이 나가는 패킷보다 많아지면 어떻게 될까? 삭제되면 안되므로 임시 저장해야 한다. 이를 위한 `buffer(혹은 queue)`가 필요하다.
    - 다시말해, 패킷을 검사하고, 큐의 제일 뒤로 넣어 순서를 기다려야한다. 이를 `queueing delay`라고 한다.
    - 이제 큐에서 빠져나와 나가려 한다. 데이터도 크기가 있겠죠? 1번 비트부터 100번 비트가 있다고 하자. 첫 비트가 나간 시간부터 마지막 비트가 나갈 때까지의 시간이 `transmission delay`다. `패킷 크기/link bandwidth`. 왜냐하면 bandwidth이 크면 한번에 큼직하게 내보낼 수 있으니.
    - `propagation delay` : 마지막 비트가 링크 위에 올라와 다음 라우터에 도달하는데 걸리는 시간. 이는 단순히 전자기파의 속도니 `빛의 속도`다. 즉, `링크 길이/빛의 속도`. 빛의 속도는 건드릴 수 없으니 링크 길이가 결정한다.
- delay는 작을수록 좋다. propagation delay는 어쩔 수 없으니 나머지를 보자.
    - processing delay는 라우터 좋은 것을 사서(성능을 개선) 빠르게 처리하게 하자.
    - transmission delay는 케이블의 bandwidth를 늘리면 줄어든다.
    - queueing delay는 제일 골치아프다. 이 크기는 접속한 인원들의 수가 결정하니 사람들의 생활 패턴과 관련 있어서 어쩔수 없다. 그래서 `가장 큰 문제`다.
        - 만약 queue의 크기보다 더 많이 들어왔다면? 방법이 없다. 큐에서 일부 정보가 유실된다. 즉, 사용자가 몰리면 `패킷이 유실(packet loss)`된다. 인터넷 패킷 유실의 90%는 여기서 일어난다.
        - 잠깐만? TCP는 reliable하다고 했는데? 네트워크 상황 안 좋으면 떨궈지잖아? 그 때는 `재전송`한다. 이 일은 `처음 보낸 쪽`에서 재전송한다(직전 라우터나 그런애들 말고). 사실상 네트워크 엣지에 모든 지능이 있고, 나머지 라우터들은 그냥 무지성(dumb core)으로 전달만한다.