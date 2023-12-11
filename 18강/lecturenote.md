# 무선 이동 네트워크1
- link layer의 연속이다.(지금까지는 유선, 이제부터는 무선)
- 이제 무선 link인 상황이 많아졌다.
- medium은 `공기`다. 충돌은 어떻게 처리하는가(`MAC protocol`)?

## 개요
- Wireless, Mobility 이렇게 두 부분으로 나누어 이야기할 것이다.
- `선이 없으면 wireless`(노트북으로 강의실에서 인터넷 사용). `네트워크 관점에서 이동하면 mobility`(즉, 다른 AP로 이동하거나 다른 네트워크로 이동하는 것, 예시로 걸어가면서 스마트폰 하기).

## Elements of wireless network
- 여러 기기가 있고, `Access Point`에 연결되어 있다.
- 네트워크를 갈 때, `첫 hop은 무선`이다. 근데, 그 뒤는 유선이겠죠? 즉, 한 hop의 문제, link layer의 문제.
- 전송 반경, Data rate(데이터 속도)에 따라 나눌 수 있다.
    - Wi-fi는 전송거리는 `짧다`. 속도는 최신으로 갈수록 점점 `빨라진다`. 그래서 주로 `실내`에서 사용한다.
    - cellular network에서 사용하는 3G, LTE, 5G는 전송거리가 길다.
- infrastructure(예 : AP)가 있냐, 없냐. single hop이냐 multiple hop이냐로 나뉠 수 있다.
    - 우리가 주로 보는 것은 `infrastructure 존재, single hop`.
    - wi-fi는 AP, cellular이면 base station 존재.

## Wireless Link 특징
- 유선 인터넷에서는 cable을 통해 신호가 전달된다. 케이블을 통해 외부로부터 `보호 받으며` signal이 안정적으로 전송된다.
    - 그래서 전송 거리에 따른 신호 세기가 `일정`하다.
- 무선 인터넷은 `보호 받지 못한다`. `간섭도 많다`. 매체가 open되어 있기 때문이다.
    - 그래서 전송 거리에 따른 신호 세기가 `exponential하게 감소`한다.
    - wi-fi는 2.4GHz를 사용한다. 이 주파수를 여러 다른 기술도 공유해서 문제가 크다. 전자레인지도 여기라서 전자레인지 옆에서 wi-fi가 안되는 경우도 있다.

### 예시
- 무선 host A, B, C가 있다고 치자. 각 wi-fi는 100m까지만 전송되고, 각 host는 일직선상에 80m 거리로 위치한다.
- A가 B에게 무언가를 보내고, C가 B에게 무언가를 보내고자 한다.
    - `A의 신호는 C가 감지 못하고, C의 신호는 A가 감지 못한다`. 하지만 `충돌이 일어나고 있다`. Carrier sense가 `불가능`하다. 이를 `hidden terminal problem`이라고 한다.
    - 기존 CSMA/CD로는 이를 감지할 수 없으니 해결할 수 없다.
- `Collision Detection이 불가능`하다.
    - 내가 무언가를 이야기할때, `내 신호가 너무 커서` 다른 신호가 noise로 들려(`잘 안들려서`) collision detection이 불가능하다.

## Wi-fi(IEEE 802.11)
- Wireless Fidelity.
- 과거에 음원과 관계된 Hi-fi가 있었다. 원음에 가깝게 재현하는 기술을 의미한다. Fidelity가 `원본에 충실하게`라는 의미다.
- 2000년대 초반, IEEE 802.11을 좀 친숙하게 하려고 `무선 링크임에도 유선 링크처럼 재현한다`라는 의미로 Wi-fi라고 한다.
- 시간이 지날수록 `Data rate이 발전하며 빨라지고` 있다.
    - 11n 버전은 안타나 여럿 두어 병렬 전송 가능하게 해서 빨라졌다.

### 구조
- `AP`가 있고, 여기에 이더넷 케이블이 꽂혀 있다.
- 여러 AP가 switch로 연결되고, 이게 하나의 router로 연결된다.

### AP는 어떻게 스캔하는가?
- AP는 `주기적`으로 자신의 정보를 담아서 `broadcasting`한다(`Becon frame`). 초당 10번 정도 주기로 한다.
- Becon에는 AP의 이름과 MAC 주소, signal세기가 담겨 있다. signal 세기를 보고 접속한다.
    - 이를 `passive scanning`이라 한다. host는 가만히 있고, AP가 주는 정보로 접속하기 때문이다. 반대로 active scanning이 있는데 잘 안쓰인다.

### MAC protocol
- 아까 예시를 생각하자, `A, B 사이에 AP`가 있다.
- 기존 CSMA/CD는 사용할 수 없다. Carrier sensing이 잘 안되고, Collision Detection도 안된다. 충돌이 나는데 충돌이 나는지 모른다. 충돌이 난걸 감지해서 충돌이 안날 때까지 데이터를 재전송하는 매커니즘인데 충돌이 난걸 모른다.
    - 기존 CSMA/CD는 ACK이 필요 없었다. 충돌을 스스로 감지할 수 있었기 때문이다.
- 이제 무선 네트워크는 충돌이 난지 모르니, `ACK이 필요`하다.
    - TCP ACK과 다르다. `TCP는 end와 end 사이의 ACK`이고, wi-fi는 `link layer에서 한 hop 사이의 ACK`이다.

## CSMA/CA(Collision Avoidance)
- 보낼 데이터가 있으면 일단 `Carrier Sense`를 한다. 있으면 `random backoff`를 하고 나중에 보낸다.
    - 이 때, carrier sense는 `DIFS 시간`만큼 한다.
- 그 후, 데이터를 보내고 `ACK를 받는다`. receiver는 SIFS만큼의 시간을 추가로 기다린 후 ACK을 보낸다. ACK을 받았을 때만 제대로 받았다고 생각하고, `못 받았다면 재전송 준비`한다.

### CSMA/CD와의 차이점
- CSMA/CD는 충돌을 감지하면, 바로 `전송을 멈춘다`. 하지만, CSMA/CA는 충돌 감지를 못하므로, 이야기가 끝날 때까지는 `전송한다`.
    - CSMA/CA가 당연히 충돌 발생 시 낭비되는 시간이 크므로 `피해가 더 크다`.
    - 충돌을 피하기 위한 동작은? 일단 ACK을 받으니 언젠가는 해결이 되나, 피해가 크니 좀 줄이고 싶다.

### RTS-CTS exchange
- `RTS`(Ready To Send), `CTS`(Clear To Send)라는 control frame을 이용해 줄인다.
    - RTS는 sender측에서 "나 지금 보낼 것 있어"라고 알리는 것. CTS는 receiver측에서 "어 보내."라는 식으로 알려주는 것.
    - 문제는 데이터를 바로 전송할 때, 충돌나면 날아간다는 것이다. 그래서, `보내고자 하는 큰 데이터가 아니라 일단 조그만 데이터를 먼저 보내보는 것`.
- 예시) A - AP - B 순으로 있다.
    - B가 AP로 무언가를 보낼 때, carrier sense를 한 후, 없으면 `RTS라는 조그마하고 special한 control frame`을 보낸다.
    - 마침 A도 AP로 무언가를 보내려고 RTS를 보낸다. AP는 두 소리가 충돌되어 `CTS를 보내지 않는다`. 잡음으로 들리니. CTS가 안 왔다면 `random backoff`를 수행한 후, 각자 따로 보낸다.
    - A의 random backoff가 먼저 끝났다. A는 RTS를 다시 보낸다. AP는 CTS를 보낸다.
        - A의 주변 아이들도 `RTS를 듣는다`. 여기에는 A가 `얼마나 큰 정보 전송할 것`인지도 적혀 있어 주변에 알려준다.
        - AP의 주변도 `CTS를 듣는다`. 지금 A가 얼마 시간동안 쓴다는 정보가 있다. `B는 그것을 듣고 기다릴 수 있다`.
        - 그 후, A는 data를 보내고, ACK을 받는다.
<br>

- 만약 AP가 A에 대한 CTS를 주변에 보내는 사이에 B가 RTS를 보냈으면?
    - 중간에 두 신호가 섞이며 noise가 된다. AP와 B의 주변의 친구들은 noise를 듣게 된다. RTS, CTS 둘 다 못 듣는다. 하지만 A와 AP 사이의 주변에 있는 애들은 CTS를 들었으니 데이터 전송은 가능해지고 A의 데이터 전송이 시작된다.
    - B의 주변 친구들은 A의 데이터 전송이 시작되었는지 모른다. 이 때, 다른 B 주변 친구(C)가 RTS를 보낸다면? collision이 일어난다.
    - 그러면 A에 ACK이 안간다. A는 다시 RTS보내고 재전송 준비.
    - 즉, 채널을 차지하기 위한 경쟁의 연속이다.
- 만약 RTS, CTS 다 잘 갔는데, 새로운 누군가 B 근처의 네트워크에 추가되고, CTS를 못 받았으니 RTS 보내면? 위와 같은 과정 반복.
<br>

- 결국 ACK 받을 때까지 재전송. 만약 이 재전송이 반복되면? `7번까지 해보고 포기`. 다음 프레임으로 넘어감.
    - 포기된 프레임은 평생 재전송 되지 않는가? 아니다. 일부 정보가 누락되어 제대로 전송 안되었으니, `TCP단에서 다시 내려와 재전송`된다.

#### SIFS
- DIFS가 10ms면 SIFS면 5ms. SIFS가 짧은 이유는 채널이 복잡해 지기 전에 최대한 빨리 ACK을 보내야 문제가 안생기므로.
- DIFS, SIFS가 존재하는 이유는 그냥 정해진 SPEC이다.

### 802.11 frame
- data부분에 IP packet이 들어간다.(제일 크다)
- CRC가 tail에 붙는다.
- 가장 중요한 것은 `address field`다. `4개`나 된다.
    - address4는 무시해도 좋다. 안 쓰인다.
    - address1, 2, 3는 항상 쓰인다.
        - address1은 Wi-fi frame 수신하는 인터페이스의 MAC address
        - address2는 프레임 전송 측의 인터페이스의 MAC address
        - address3는 이 프레임 처리할 router의 MAC address
- 예시)
    - host1 -> AP1 -> router1이면, `AP1, host1, router1순`으로 나온다.
    - AP는 신기하다. `한쪽은 무선, 한쪽은 유선`이다. 즉, 한쪽은 CSMA/CA 프로토콜, 한쪽은 CSMA/CD 프로토콜. 한쪽은 wi-fi frame이 왔다갔다, 한쪽은 ethernet frame이 왔다갔다.
    - 여기는 심지어 `link layer`의 device. network계층도 아니고 그냥 link layer만.
    - 이것을 토대로 왜 address field가 다른 frame과 다른지를 생각해보자.