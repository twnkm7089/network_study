# 전송 계층 4

## TCP Congestion Control
- 비유를 들자면 보내는 TCP에서 받는 TCP로 전송하는데, 네트워크가 복잡하게 이곳 저곳에 연결되어 있다. 우리는 데이터를 받고 싶지만, 네트워크의 상태는 모른다. 너무 보내면 터진다.
- critical한 부분은 가장 병목 현상이 크게 발생하는 곳이 될 것이다. 하지만, 우리는 거기가 어디인지 모른다.

### 3 main phases
1. `Slow Start` : 시작은 조금씩 데이터를 보내주며 데이터가 감당 가능한지 확인. TCP는 처음에는 아주 조금인 1개로 시작해, 2개, 4개, 8개, ... 이런식으로 통크게 늘려 나간다(`exponential`).
2. `Additive Increase` : 그렇게 2배씩 올리다가 threshold를 넘으면 조심해야 한다. 이때 부터는 exponential이 아닌 `linear`하게 증가한다. 이 linear한 구간이 Additive Increase다.
3. `Multiplicative decrease` : packet loss를 탐지하면 해당 위치에서 window size를 `절반으로 줄인 후` 다시 조금씩 늘리며 시작한다.
    - 왜냐하면 막혔다 싶으면 모두가 뒤로 확 빼야(절반으로 줄여야) 상황이 해결되기 때문이다.

### Maximum Segment Size : 500byte
- segment는 최대 500byte다.
- 위에서 설명한 몇 개가 바로 `몇 MSS`인가이다.
- window size는 처음 시작할 때는 slow start로 인해 1, 2, ... 조금씩 증가한다. 이는 1MSS, 2MSS, 4MSS, ...이런 식으로 증가한다.

### 대략적인 데이터 전송 속도.
- rate = CongWin(congestion window size)/RTT(Round Trip Time) bytes/sec
    - 실제로 변동 심한 것은 RTT보다는 CongWin이다.
    - Congestion window를 조절하는 것은? network.

### TCP Tahoe vs TCP Reno
- x축은 `시간`, y축은 `congestion window size`.
- TCP의 `첫 버전` TCP Tahoe의 동작.
    - 일단 들어와서 slow start를 하고 늘려간다.
    - threshold값 부터는 하나씩만 증가하게 한다.
    - 패킷이 유실되면? network congestion이라 판단하고 `slow start`부터 다시 시작. 그리고, 아까 `최대값 / 2`를 하여 새로운 threshold를 정의한다.
- `두번째 버전`인 Reno
    - 위에 다른 것은 똑같고, 만약 유실이 되면?
    - `Timer`나 `3 duplicated ACK`를 이용하여 유실을 탐지했다.
        - `3 duplicated ACK`를 이용한다는 것은 다 잘가지만 이것만 문제가 발생한 상황이다.
        - `Time out`은 그 패킷 뿐만 아니라 다른 패킷도 다 안간 상황이다.
        - 둘의 대응이 같아서는 안될 것이다.
    - 만약 `Time out`이 탐지되면? window size와 threshold는 둘 다 `절반으로 줄인다`.

### 질문
- 처음에 Threshold 설정은? 구현하기 나름.
- 조금조금 늘이다가 터지면 줄이면 좋을 것이다.

### 조금 더 생각해보자.
- 지금까지는 둘 끼리의 congestion control을 봤다. 근데, 생각해보면 네트워크는 아무나 쓰는 것이 아니다. 모든 사람들이 `독립적`으로 congestion control을 수행한다면, 모두가 `공평하게 사용하는가`?
    - 최대 R의 bandwidth 가진 네트워크가 있다고 치자. 과연 이런 방식을 통해 둘 다 각각 R/2만큼을 가지게 되는가?
    - 처음에 한명이 R만큼 쓰다가 누군가가 참여할 것이다. 최후에는 R/2가 되는가? 결론을 이야기하자면 `fair`하게 된다.
    - connection 1이 우세한 상황이었다 치자. 둘 다 늘이고 늘이다 bandwidth를 넘어버리면서 절반으로 줄인다. 이를 반복하다 보면 어느샌가 connection 1, connection 2가 R/2로 수렴하게 된다.
- 다만, 한가지 맹점이 있다. 모든 TCP가 공평하게 갖게 되나, 만약 한 쪽에서 TCP connection을 하나 더 열면? 더 많이 쓴다.
    - 즉, `TCP 연결 끼리의 공평`이므로, TCP를 여러개 연 사람이 있으면 그만큼 몫이 더 많아진다.