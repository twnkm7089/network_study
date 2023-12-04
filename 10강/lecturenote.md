# 네트워크 계층 2

## IP datagram format
- data가 TCP 혹은 UDP segment, 크다.
- 그 앞은 IP header에 해당하는 필드다.
    - IP Protocol version
    - length(패킷 전체 길이, 바이트 단위)
    - `source IP address, destination IP address`(가장 중요), 필드는 32bit크기. 32bit 주소 체계 사용중
    - checksum : 오류 check 위해
    - TTL(Time To Leave) : `라우터를 거칠 때마다` 여기의 값을 -1, 0이 되는 순간 버려진다. 잘못해서 loop를 돌게 되는 등의 문제를 해결하기 위해.
    - upper layer : data에 들어가는게 TCP인지 UDP인지 표시.
    - identifier, flags, fragmentation offset은 다음 시간에...(셋이 같이 쓰임)
- IP header를 모두 합치면 20bytes 나온다. TCP header 역시 20bytes 나온다. 즉, 기본적인 application message에 network 계층까지 내려오면 `40bytes의 overhead`가 붙는다.
    - 인터넷에는 많은 수의 40bytes짜리 패킷이 있다. 왜? 바로 `ACK만 담은 패킷`.

## IP Address(IPv4)
- 32bit 주소체계. 이론적으로 2^32개의 IP 주소가 가능하다.
- 8bit씩 끊어서 10진수로 바꾼 것이 우리가 흔히 읽는 방식. (xx.xx.xx.xx), (xx는 최대 255), (기계는 당연히 이진 bit로 보겠죠?).
- `host에 들어 있는 네트워크 인터페이스 자체`를 지칭하는 주소다.
    - 컴퓨터에 여러 네트워크 인터페이스 카드 꽂는다? 그러면 여러 개의 IP 주소 가질 수 있다. 그래서 host의 주소라고만 하면 부족하다.
    - 여러 네트워크 인터페이스 카드를 꽂는 대표적인 예시로 `라우터`가 있다.

### IP 주소의 배정 방식
- 가장 단순한 주소 배치 방식 : 그냥 랜덤하게 배정한다.
    - 이럴 경우, 라우터 안의 `forwarding table이 엄청 복잡해진다`. host가 죄다 뒤죽박죽일 것이므로.

#### 현재의 배정 방식 : `계층화`
- 앞부분은 `네트워크 ID`, 뒷부분은 네트워크 내의 `호스트 ID`. 같은 네트워크면 같은 네트워크 아이디를 가진다.
    - `네트워크 ID가 24bit이면 xx.xx.xx.xx/24` 같은 식으로 표시해준다. 네트워크 ID를 `subnet ID`, 혹은 `prefix`라고 하기도 한다.
- `subnet mask` : 이 주소중 어디까지가 prefix인지 알려주는 것. 해당 부분까지 1, 나머지는 0이다.
- prefix가 24bit이면 뒷부분은 8bit니, 256개의 주소 가진 네트워크이다.
- 이 방식이면, 같은 네트워크 내에서는 같은 prefix를 가지므로 `forwarding table`이 단순해진다. 또, `새로운 host를 추가`할 때도 뒤쪽만 추가로 assign해주면 되니 더 수월해진다.

### 주소 배정 방식의 역사
- 인터넷의 구성 요소는 네트워크다. 여러 네트워크를 연결해 놓은 것이다. 각 네트워크는 자기 자신의 prefix를 가져야 한다. 하지만, 그 크기는 다를 것이다.
- Classful Addressing
    - IP주소에 class를 나누어 놓았다. class A, B, C 같은 방식이다.
    - class A에 해당하는 경우, `/8`. 앞은 `0으로 시작`.
        - 이 주소를 네트워크가 배정 받으면, prefix가 8bit니, host는 2^24개까지 가능하다. 하지만, 이 주소를 가질 수 있는 기관은 128개 기관이다(앞이 0으로 고정이니 실질적으로 7bit만 가용 가능).
    - class B는 `/16`. 앞은 `10으로 시작`.
        - 2^16개의 host, 2^14개의 기관에 배정 가능.(16-2)
    - class C는 `/24`. 앞은 `110으로 시작`.
        - 2^8개의 host, 2^21개의 기관에 배정 가능.(24-3)
    - class A는 좋아보이나, 128개 기관만 가능하니 차지 불가. 심지어 크기는 너무 커서 문제. 반대로 class C는 여럿이 할당 받을 수 있으나 크기가 너무 작다.
    - 과거의 방식이 너무 비효율적이다. 1000개의 host를 위해 C는 작고, B는 크다.
    - 90년대 중반에 class가 없는 주소공간 배정으로 넘어간다.

- `Classless Inter-Domain Routing(CIDR)`
    - 이제 `원하는대로 유연하게 prefix 사이즈를 조절`해 원하는 크기의 주소를 배정 받을 수 있다.
    - 이 덕분에 forwarding table에 들어갈 주소 수도 적어져 크기도 작아졌다.

### forwarding
- prefix를 forwarding entry에 넣어 찾는다.

#### Longest Prefix Match Forwarding
- 예를 들어, destination address가 201.10.6.17이다.
    - 이제 forwarding table을 보며 entry를 매칭한다. '201.10.0.0/21'과 '201.10.6.0/23'이 매칭된다. 이렇게 여러개 매칭된 경우, 가장 구체적으로 매칭된 것, 즉, `prefix의 크기가 가장 긴` 201.10.6.0/23을 고른다. 이것이 `longest prefix match frowarding`이다.

### 정리
- 현재 IP 주소는 32bit 공간을 쓰고, prefix부분과 host ID 부분으로 나뉜다.
- CIDR방식을 사용해 원하는 대로 유연하게 network size에 맞는 forwarding table을 작성한다.

## Subnets
- 같은 서브넷 ID, 즉, `같은 prefix를 가진 디바이스의, 인터페이스의 집합`.
- `router를 거치지 않고 접근 가능`한 host들의 집합.
- router도 IP주소를 가지고 있다. `router의 각 interface도 IP 주소를 가지고`, `연결된 네트워크와 subnet`을 이루고 있다. 즉, router는 여러개의 subnet에 걸쳐 있는 존재이다, 교집합니다. 교집합이니 여기를 통해서 다른 네트워크로 갈 수 있는 것이다.
- `router와 router간의 연결`도 하나의 subnet을 이루고 있다.

## IPv6
- 기존 IPv4는 2^32개의 host, 즉, 약 40억개가 가능하다.
- 70년대 처음 나올 때는 문제 없었지만, 90년대 인터넷 상용화 이후로는 주소 공간이 고갈될 것으로 예견되었다.
- 1996년에 IPv6가 탄생하게 된 계기다.
- 주소공간이 `128bit`로 늘어났다. 전 세계 해변의 모래알보다 많다.
    - 각 부분마다 16진수 네 개로 8부분을 사용해 나타낸다.
    - AAAA:BBBB:CCCC:DDDD:EEEE:FFFF:1111:2222
- 그럼에도 2015년 기준으로 아직 IPv4를 사용중이다. 잠깐? 주소 공간이 고갈될 거라면서? 이미 고갈되었다. 그런데 어떻게 버틴 것일까? 주소 공간을 공유하며 재활용하고 있다. 그 트릭이 NAT다.

## Network Address Translation(NAT)
- 인터넷 상에서 모두가 고유한 IP 주소를 가지기는 어렵다.
1. 네트워크 내부에서는 `네트워크 내부에서만` 서로 유일한 IP 주소를 사용한다. 이 패킷이 외부로 나갈때는 NAT 기능을 하는 `gateway router`에서 IP를 공통된 gateway router의 IP로 바꾸어준다. 물론 이 주소는 `세계에서 유일`하다.
2. 외부로 나가는 패킷의 source IP address는 이 gateway router의 IP로 바뀐다.
3. 돌아올 때는 반대로 destination IP address가 gateway router로 설정되고, router에서 받으면 알맞은 host의 IP 주소로 변환되어 해당 host로 보내진다.
- gateway router가 어떤 host로 온 것인지 알 수 있는 방법은?
    - 나갈 때마다 NAT translation table에 WAN side addr, LAN side addr에 기록된다. 이 addr에는 IP주소와 port번호가 기록된다. 두 경우의 port 번호는 다를 수 있다.
    - `돌아오는 패킷에 적힌 port 번호를 보고`, 해당 host를 찾는다.
    - 근데 잠깐? `router는 network계층까지만` 가능한데, `port 번호는 TCP, 즉, transport 계층의 header`에 있다. 더 상위 계층의 header를 읽으려면 data부분을 보아야 한다. 이는 `layer violation`이다.
        - 또, IP주소는 host를 찾아갈 때, port number는 host 내부의 프로세스를 찾아갈 때 사용해야 한다. 근데, host 찾을 때 port number를 써버리면, `그 후에 프로세스는 어떻게 찾아가냐`?
        - 그래서 NAT network 내부에서는 `서버를 운영할 수 없다`. 포트 번호를 사용해 프로세스로 이동이 불가능하므로.