# 네트워크 계층 3

## 지난시간 : TCP connection을 유지하느냐 안하느냐로 문제가 달라지고, source, destination IP, MAC이 유지되어야 유지된다.

## Wi-fi : advanced capabilities
- AP와 wi-fi host 사이의 무선 channel의 quality는 계속 변한다. AP와 mobile기기 사이의 `거리에 의해 좌우`되고, `가까울수록 좋다`.
    - 이 때 사용되는 지표가 `SNR(Signal to Noise Ratio)`다. `잡음 대비 신호가 비율이 높을수록 좋고`, 값이 클수록 좋다.
- `전송하고자 하는 디지털`인데, 무선 상에서 `둘 사이에 전송되는 신호는 아날로그`다. 고로, 이 아날로그 신호에도 약속이 있다. 어떤 식으로 데이터를 실을지(`encoding`)는 약속되어 있고, `high coding rate`를 이용해 한정된 공간에 더 많은 데이터를 실을 수 있도록 할 것이다.
- 만약 `오류가 커지면` `작은 전송률`(coding rate)를 가진 것을 이용할 것이다.
    - 왜냐하면 coding rate가 높은 기술을 사용할수록 BER(비트당 에러 확률)이 높아진다. 에러가 생기면 에러검증 과정에서 ACK이 안오므로 재전송률이 커진다.
    - coding rate를 낮춘 기술을 사용할 경우, SNR이 줄어들지만 BER이 낮아진다.

### power management
- 컴퓨팅에 쓰는 전력과 비교해 `무선통신을 할 때 소모 전력`이 10배다.
- 그래서 RTS-CTS 교환 예시에서 통신 하는 경우를 제외한 나머지 host들이 대기 상태에 들어갈 때, `wi-fi adapter의 전원을 꺼둔다`.

## Cellular network
- 전체 coverage를 `cell로 나누고`, 각 cell을 담당하는 `base station`을 세워 구성한 네트워크다.
- 여기서 쓰는 MAC protocol은? `첫 hop만 무선`.
    - wi-fi 때의 random access 방식과 달리 channel partitioning 방식을 사용한다.
    - combined TDMA/FDMA : 주파수, 시간 별로 나누었다.
    - CDMA : `3G 이후`의 방식. 주파수, 시간을 분할하지 않고, 각 사용자들에게 각기다른 `code`를 준다. 이 code에 대한 연산을 진행하면 `나한테 오는 신호는 증폭, 다른 신호는 noise처리되어 감소`한다.
- cellular network는 `data rate으로 세대`가 나누어진다.
    - LTE는 원래 1Gbps에 도달하지 못해서 4G 기술이 아니었음. 그래서 3.5G였는데, LTE-A(LTE Advanced)라는 기술이 나오고서야 비로소 4G가 되었다.
    - 4G 시장 차지를 위해 LTE를 들고 온 세력과 Wibro를 들고 온 세력이 있었다. 결국 LTE가 이겼다. LTE가 이긴 이유는 EU를 중심으로 뭉쳐 세력이 더 컸고, 유럽의 기존 2G망에서 base station 소프트웨어 업그레이드만 하면 사용 가능해서였다.

### 구조
- 사용자가 있고, 무수히 산재한 base station이 있고, 그걸 하나로 묶는 SGSN이 있고, 최상위에 GGSN이 있고 그게 인터넷이랑 연결되고... 이런 형식의 `피라미드`가 존재한다.
- 즉, 우리 스마트폰으로 인터넷을 한다는 것은 IP주소가 배정받았다는 것이다. 이를 GGSN이 준다. 당연히 GGSN으로 이루어진 subnet 내부에서 다른 IP를 주겠죠?(수 많잖아)
- GGSN은 수가 엄청 적다.
- `GGSN subnet 내부`의 protocol은? cellular network 내부의 고유한 protocol을 다룬다. 이건 대학원 영역이다.

## Mobility
- 사용자가 네트워크를 넘나드는 과정에서 TCP connection이 안 끊기는 방법은?
    - 아직 현대 네트워크에서 `구현 안됨`. 개념일 뿐.
- mobility의 정의 : mobility는 아얘 없는 상황(가만히 한 자리에 있음), 중간 상황(인터넷 하다 중간에 멈추고, 가서 새로운 연결, TCP connection 유지 불필요, DHCP로 처리 가능), `높은 상황(계속 TCP connection 유지하며 다른 network 이동)`
- 우리가 서울에서 부산갈 때 계속 영화 볼 수 있는 이유는? base station은 바뀌어도 `같은 network` 소속이기 때문이다. 이동하면서도 유지되니 `이동통신`이다.
- 하지만 우리는 `네트워크를 넘나들고 싶다`.
    - 계속 이동하는 기기와 `어떻게 연결`할 것인가?
    - 연결을 맺은 후 `어떻게 유지`할 것인가?

### 용어
- mobile host : 각자 자기자신의 permanent address를 가짐. 어디 가도 유지.
- home network : 각자의 permanenet address를 가진 네트워크 존재.
- home agent : home network에 속한 모든 host를 관리. GWR에 있는 tracker를 생각하면 된다.
- visited network : 새로 방문한 네트워크
- care-of-address : 기존 mobile host의 address는 유지하되, 새로운 네트워크에서의 address를 추가로 배정 받는다.
- foreign agent : 방문한 네트워크의 agent

### 등록
- 모바일 네트워크가 다른곳으로 이동했다. foreign agent에 이를 알린다. foreign agent는 home agent에 현재 여기 있다는 것을 알린다.
- 누군가 해당 host에 접근 원하면 permanent address를 보고 home agent에 요청 보내고, home agent는 이를 보고 foreign agent로 forwarding 시킨다(`indirect routing`).
    - 장점은 트래픽 보내는 입장에서는 `편하다`. 단점은 멀리 돌아가므로 `오래 걸린다`.

### 유지
- 커넥션을 유지한 상태로 이동하고 싶다면? 도착한 곳의 foreign agent에게 연락해 home agent에 관련 정보를 갱신해준다. 그렇게 하면 connection을 유지해준다.

### direct routing
- 이번에는 home agent에서 바로 forwarding해주는 것이 아니라 보내는 측에 home agent가 현재 위치를 되돌려주어 해당 정보를 바탕으로 연결을 맺는다.
    - 직접 연결 되어 빠르나, 할 일이 많아진다.
- 유지의 경우, 이전의 foreign agent가 다른 foreign agent에 forwarding을 하는 방식이다.

## 위의 개념들을 실제로 구현 안되었다. Mobile IP 같은 표준이 90년대 중반에 나왔으나 사용되지 않고 도태되었다.