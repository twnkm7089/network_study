# 네트워크 계층 3

## Network Address Translation(NAT)
- IP주소의 재사용(여러 사람이 같은 주소 사용)
- 같은 네트워크 내부에서는 그 `네트워크 내부에서만 유일한` 고유한 IP 주소 부여
    - 외부로 나가면 유일하지 않으니 문제가 발생할 수 있다.
    - gateway router가 보고 있다가 내부에서 나가는 패킷의 source 주소를 자기 자신의 IP 주소로 바꿔주는(이건 세계에서 유일) 역할을 한다.
    - 바깥에서 돌아오는 패킷은 gateway router에서 변환해서 내부 네트워크의 해당 위치로 보내준다.

### 좀 더 자세히
- 각 인터페이스가 IP주소를 가진다.
- 같은 subnet에서는 같은 prefix를 가진다.

1. host가 외부로 패킷을 보낸다.
2. gateway router에서 source address를 바꿔서 subnet 바깥으로 내보낸다.
3. response가 gateway router로 돌아온다.
4. gateway router가 원래 host로 보내준다.

#### 문제점
- `layering violation` : router(NW 계층 장비)가 port번호(Transport 계층인 TCP header에 위치, NW계층에서는 data영역)를 보고 고치기까지 함.
    - 원래 IP주소로 host를 찾고, port번호로 프로세스를 찾는데, 앞에서 사용해서 `프로세스를 찾을 수 없다`.
    - webserver의 경우, NAT translation table에 미리 써놓지 않은 이상 `해당 위치로 패킷을 받을 수 없다`.

- 근본적으로 IPv4는 `주소공간부족`과 `보안 문제`에 시달린다.
    - 해결을 위해서는 `근본적으로 IPv4에서 다음 세대로 이행`할 필요가 있다.
    - 하지만 대안으로 나온 IPv6도 1996년에 디자인된 옛날 디자인이고 미래에 어떤 요구사항이 있을지를 예측할 수는 없다. IPv4도 70년대에는 문제 없을 것으로 보였었고, 한번 정착된 생태계는 쉽게 바뀌지 않는다.
    - 현재는 IPv6로 갈지 아얘 다른 future internet으로 갈 지가 논쟁거리이다.

## Dynamic Host Configuration Protocol(DHCP)
- A의 IP주소는 192.168.1.147이고, 서브넷마스크는 255.255.255.0, router는 192.168.1.1, DNS는 192.168.1.1이다.
    - 이 정보는 A가 인터넷을 하기 위해 알고 있어야 할 가장 기본적으로 알고 있어야 할 정보다.
    - IP주소와 subnet mask로 보아, 현재 A가 속한 subnet의 주소는 192.168.1.0/24
    - router의 주소는 A가 `바깥에 패킷을 보내기 위해 무조건 거쳐야 하는 router의 주소`다. 패킷을 외부로 보내려면 이 router로 무조건 보내서 forwarding하게 해준다.
    - 주소를 보자, router의 주소는 `해당 subnet의 첫번째 주소인 것을 확인할 수 있다`.
    - DNS 주소는? 웹 브라우저에 www.naver.com을 쳤다고 보자. 우리는 이 주소를 IP 주소로 변환해야 하고, 그러기 위해서는 `local name server의 주소`를 알아야 한다. 그 주소가 이 주소다.
- DNS 주소를 몰라도 직접 IP 주소를 치면 인터넷은 가능하다. 좀 불편할 뿐, 나머지는 무조건 알아야 한다.
- 이 정보들은 누가 적었을까? 바로 `DHCP`다.
    - 노트북 뚜껑을 열었을 때 `제일 먼저 configure`해주는 프로토콜.
    - 어디서든 `host를 동적으로 configure`해준다. 새로운 곳에서는 새로운 정보가 필요하므로.
    - 물론, `static IP`라고, 자기자신의 `고정된 IP`를 가지는 경우도 있다. 그 때는 스스로 이걸 작성해 놓을 수 있고, DHCP가 필요 없다. 하지만 대다수는 `dynamic IP, 즉, 계속 바뀌는 IP`를 사용한다.

### 고정 IP사용 시?
- DHCP는 필요없다.
- 하지만, 이 정책을 사용하면 고정 IP를 사용하는 집단은 그렇지 않은 집단보다 필요한 IP 개수가 많다. dynamic IP의 경우 `필요할 때마다 active하게 남아있는 IP에서 하나씩 할당`하고, `끝나면 회수`하는 방식으로 운용할 수 있기 때문이다. Address를 유연하게 사용 가능하나, 회수를 해야 하니 DHCP가 필요하다.
- 예시)
    1. 223.1.2.0/24 subnet에 A가 들어와서 노트북 전원을 켰다. A에게는 아직 아무 정보가 없다.
    2. A는 `DHCP discover message`를 보낸다. source address를 0.0.0.0, destinationn address를 255.255.255.255로 우선 초기화한다(모르니까). 이렇게 `destinationn address가 255.255.255.255면 broadcast를 해서` 연결된 모든 network로 보내진다.
    3. 이 DHCP discover message는 `DHCP server를 제외한 나머지 host는 무시`하게 된다. 어떻게? `DHCP server만 목적지에 해당하는 해당 port를 열고 있고`, 나머지는 닫고 있다.
    4. DHCP server는 `DHCP offer message`를 보낸다. 아직 A의 이름이 없으니 `broadcasting`으로 보내고, 본인의 주소를 source address로 보낸다. Broadcasting으로 보내나 해당 port는 A만 열고 있으니 A만이 의미있게 받아들일 수 있다. `yia address에 "이 IP주소 어떠냐? 이 IP주소로 이 정도 시간 할당해 주겠다."`는 의미로 주소 정보를 보낸다.
    5. A가 이를 보고, 이 offer가 마음에 드는지 응답한다(`DHCP request`). 마음에 들면 transaction ID를 1 증가시킨 후, DHCP discover 때와 동일한 방식으로 응답한다.
    6. DHCP server가 이를 받아 `DHCP ACK`을 A로 보낸다. 방식은 DHCP offer때와 같고, A가 이를 받으면 드디어 A에게 IP주소를 포함한 아까의 정보가 생긴다.

#### Offer로도 충분하지 않나?
- 어떤 경우, `DHCP server가 여러개` 있을 수 있다. A가 `여러개의 offer`를 받으면, 그 중 하나를 선택해야 한다. 이를 위해 추가적인 DHCP request, DHCP ACK이 있다.

#### 왜 DHCP request의 destination address도 broadcast인가?
- DHCP server가 여러개일 수 있다. 간접적으로 `다른 DHCP server`에 offer 관련 정보를 알려주기 위해서다.

#### 보통 DHCP server는 gateway router에서 돌아가고 있다. local name server도 gateway router에서 돌아가고 있다.
- 또, DHCP server, local name server는 application layer 영역에서 돌아가고 있다. 원칙적으로 gateway router는 forwarding만 해야 하지만 이런 일들도 추가로 한다.
- 또, gateway router는 NAT 기능도 수행한다.
- firewall의 기능도 수행한다.

### gateway router의 실제 사례
- 우리가 예를 들어 SK broadband로부터 인터넷 회선 하나를 쓴다고 보자, 그러면 사실 SK broadband는 우리한테 IP주소를 하나 할당해준 것이다.
- 집에 공유기가 있다. 이 공유기가 gateway router다. NAT, DHCP 등의 다양한 기능들을 해준다.
- 공유기에 연결된 각 기기의 IP 주소는 다르나, 나갈 때는 같아진다.
<br>
- 그런데, 공유기로 나간 IP도 세계 유일일까?
    - 모른다. 사실 이 공유기의 IP도 더 큰 NAT 네트워크 내부의 IP일 뿐일지도...

### 참고
- 나중에 배우겠지만, link layer에서는 MAC주소를 쓴다.
- destination IP주소가 255.255.255.255면 broadcasting이다. 이게 MAC주소로 변환될 때는 모두 bit 1 (FFFFFFFFFFFF)로 변환된다.

## IP header에 있던 16-bit identifier, flags, fragment-offset의 의미

### IP fragmentation, reassembly
- IP 패킷을 보냈다고 하자. header에 적힌 length는 4000 bytes다.
    - 보낼 수는 있지만, link layer에서 각 link로 보낼 수 있는 최대 크기 MTU(Maximum Transfer Unit)는 link 종류마다 다르다.
    - 보내던 도중 어떤 link에서는 최대 1500 bytes씩만 처리 가능하다고 하자. 그러면 어떡해야 할까? 처리할 수 있는 사이즈보다 큰 패킷이 오면 독립적인 사이즈의 frame으로 분리한다.
    - 나중에 합쳐서 원래 패킷으로 복원한다.
    - 이 작업을 해주는게 IP header의 해당 부분이다.

- 예시
    - length : 4000bytes, ID = x, flag는 fragment된게 뒤에 있는지 여부(현재는 없으니 0), offset은 전체 패킷 중 여기가 어디부터인지(지금은 0).
    - 1500MTU짜리 link를 만나서 분리된다.
        - 각 length는 전체 패킷 길이다. header는 20 bytes였으니, data는 원래 3980 bytes다.
        - 원본 데이터는 3980 bytes, 일단 첫 패킷에 이 중 0~1479가 들어간다. 두번째 패킷에는 1480~2959까지 들어간다. 즉, offset은 1480/8=185로 해준다(`8로 나누는` 이유는 bit 절약).
        1. length=1500, id=x, flag=1, offset=0
        2. length=1500, id=x, flag=1, offset=185
        3. length=1040, id=x, flag='0'(뒤에 더 없음), offset=370

### 만약 분리된 패킷 중 중간에 하나가 유실되면?
- reassemble이 안된다. -> 위의 layer로 못 올린다 -> 전체 패킷 유실 -> `TCP timer 터지면서 재전송`.