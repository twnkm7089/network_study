# 5강 전송계층 1

## 복습) Reliable Data Transfer 복습
- unreliable한 channel 환경에서 발생하는 packet error, packet loss
- packet error 해결을 위한
    - error detection, feedback, retransmission, sequence number
- packet loss 해결을 위한
    - timeout

- 하지만 이 프로토콜은 성능이 너무 안좋다.
    - `Utilization` : Sender가 전체 시간 중 네트워크를 사용하는 비율. 높을수록 좋다.
    - Transmission Time = L/R (패킷 길이 / 전송률(bit per second))
    - 이 프로토콜은 U = (L/R)/(RTT+L/R). RTT는 Round Trip Time(데이터 패킷이 전송되는데 걸리는 시간과 ACK이 다시 도달하는데 걸리는 시간의 합). Utilization이 너무 작다.

- 우리는 한번에 쏟아붓고 한꺼번에 피드백 받는 pipelining을 지원하는 것이 더 좋다.
- 만약, 한번에 3개씩 보내면? Utilization은 3*(L/R)/(RTT+L/R). 3배다. (첫번째 ACK이 도착하자마자 다른걸 새로 보내니.)

## 오늘의 목표
- Pipelining을 가능하게 하는 go-Back-N, selected repeat에 대해 알아보자.

## Go-Back-N
- 이제 한꺼번에 많은 패킷을 쏟아 붓는다(selected repeat도 마찬가지.). 근데, 얼마나 보낼지의 기준이 필요하다.
- 이 기준이 바로 `window`, window size만큼은 feedback 없이 한번에 보낼 수 있다.
<br>
- 예를 들어 window size = 4면, 0, 1, 2, 3번 패킷은 한꺼번에 보낼 수 있다.
    - 이 때 받는 ACK은 `cumulative ACK`이다. 예를들어, ACK 11은 '나는 11번까지는 완벽하게 받았다. 12번 패킷을 달라'라는 의미이다.
    - 보낼 때는 각각의 패킷에 각각의 타이머가 달려 있다. 만약 0번 패킷의 타이머가 터지면? window내의 0~3번 패킷 모조리 다시 전송.
- 그럼 Receiver는?
    - Receiver에는 버퍼도 없고 아무것도 없다. 오직 자기가 받아야 하는 패킷의 sequence number만 있다.
    - 예를들어, 0번 패킷을 받으면, ACK 0 보내고 1번 패킷을 기다린다.
        - 근데 만약, 중간에 한번 꼬여서 패킷 1번이 도착하기 전에 2번이 도착하면? 내가 기다리던 것 아니니까 무시하고 ACK 0(0번까지 제대로 받음)을 전송한다.
        - 1번 패킷이 와야만 비로소 ACK 1을 보내고 2번을 기다린다.
        - 근데, 아까 2번 버렸죠? 그래서 ACK 1 보내고 2번 기다리는 상태에서 멈춘다.
- 즉, `Go-Back-N에서는 Sender가 사실상 다 한다`. Receiver는 무식하므로.
<br>
- 예시)
    - window size = 4, 0~3을 보냈다.
    - receiver가 0, 1을 잘 받아 ACK 0, 1을 보냈다. 근데, 2번 패킷이 loss된 상황에서 3번 패킷이 왔다. 그래도 receiver는 무시한다. 2를 받아야 하니, ACK 1을 다시 보낸다.
    - 제대로된 ACK이 도착하면 window는 한칸씩 전진한다. ACK 0이 도착해서 window는 1~4가 되었고, 4번이 전송된다. ACK 1이 도착하면 window는 2~5가 되고, 5번이 전송된다.
    - ACK 2는 도착하지 않았다. 그러면, 2번 패킷의 timer가 터지고 2~5번이 모두 재전송된다.
    - 그리고 다시 잘 전송되어 ACK을 받으면 계속 전진한다.
- 다시 예를 들어 0번 패킷부터 보내는 도중, 6번 패킷이 유실되는 상황이라 가정하자.
    - ACK 5까지는 잘 도착하나, 7, 8, 9번 패킷이 도착할 때마다 ACK 5를 다시 보내고, 6번 패킷의 타이머가 터지면 다시 싹 다 재전송한다.
    - 결국 패킷이 나가는 도중, 유실되면 window size에 해당하는 `N개만큼 다시 돌아와서 재전송된다`. 그래서 Go-Back-N이다.
- 또, window는 한번에 전송할 수 있는 양인 동시에, `sender에서 buffer에 저장하고 있어야 하는 존재다`. window 밖의 것들은 receiver가 받은 것을 확인해서 저장할 필요 없다. 하지만, window 내부의 것은 확인이 안 되어서 저장해 두어야만 한다.

### 근데, 너무 비효율적이다. 패킷 하나 유실되었다고 window size만큼 재전송 하기에는 너무 과하다. receiver가 일 좀 해줘라.

## Selective Repeat
- Go-Back-N은 유실되면 N개를 다시 보냈다면, `Selective Repeat는 유실된 것만 selective하게 재전송 해주겠죠`?
- ACK은 cumulative ACK이 아니다. ACK 11은 11번을 잘 받았다는 의미이다. 11번까지가 아니라.
- 이제 receiver가 일한다. 문제 없이 도착한 패킷은 buffer에 저장한다. 예를들어, receiver가 `2번 패킷을 기다리고 있지만` 3번 패킷이 들어오면 일단 buffer에 3번 패킷을 저장한다. 그리고 ACK 3을 보내준다.

- 예시) window size = 4.
    - sender가 0, 1, 2, 3을 한꺼번에 보냈다. 근데 2번이 유실된다.
    - 0, 1 ACK 잘 보냈는데, 2번 기다리는데 3번이 왔네? 3번 buffer에 저장하고, ACK 3 보낸다.
    - Sender는 ACK 0, 1받았으니 window 미루어서 window는 2, 3, 4, 5가 되고, 4, 5는 추가로 전송된다(잘 도착했다고 치자). 근데, ACK 3이 왔으니 `ACK 3의 Timer는 사라진다`. 2번 타이머가 터지면? only 2번만 재전송한다.
    - Receiver가 재전송된 2번 패킷을 받았다. 드디어 `버퍼가 순서대로 찼다.` 이제 이 buffer를 application layer에 올려주고, ACK 2를 전송한다. 아까 ACK 받아서 타이머 해제되었던 분량까지 모조리 밀어버려서 이제 window는 6, 7, 8, 9로 이동한다.
    - 유실된 패킷만 전송하니 `network의 부담은 적어지나 receiver가 일을 한다.`

### 생각해 봐야할 문제: sequence number
- 지금까지는 단순히 숫자를 증가시켰다. 근데, 이 `sequence number 역시 header에 들어간 비트고, 우리는 이를 최소한으로 하고 싶다`. 어떻게 해야 할까?
- selective repeat에서 `window size가 3`이라고 해보자.
    - 만약 4개의 sequence number인 0, 1, 2, 3만 쓰면?
    - sender는 0, 1, 2를 한꺼번에 보낼 수 있다. 이제 receiver는 잘 받아 3번을 기다리고 있다.
    - 근데, ACK이 모조리 유실되었다. sender의 window 안에 있던 0, 1, 2는 재전송된다. receiver는 버퍼에 3, 0, 1을 저장할 수 있다. 0은 `duplicated packet`이나, receiver는 3 다음에 올 `새로운 패킷인 줄 안다`. 매우 큰 문제다.
    - 해결법은? sequence number를 늘리면 된다. 새로운 패킷과 duplicated packet이 구분될 수 있는 최소한의 정도로 늘리자. 이는 `window size*2`다.
#### 결론, sequence number는 duplicated packet과 새 packet을 구분하기 위한 것이니 window size와 연관이 깊다.

## 다음주
- TCP 본격
- congestion control, flow control 등을 배운다.
- TCP는 go-back-N과 selective repeat의 장점을 버무린 것을 활용한다.
    - selective repeat의 단점은 모든 window안의 packet에 timer를 달아야 한다. 실제 window size는 엄청 크다. 여기 모두 timer 달면 CPU는 감당이 안된다. 특히, 실제 컴퓨터는 여러 프로세스가 돌아가고, 이 모든 프로세스의 window에 timer가 걸리면...불가능하다.
    - 그래서, TCP는 window 대표 타이머가 있고, ACK도 cumulative이나, 이걸 응용해 selective repeat적 요소를 도입했다.