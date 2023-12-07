# 네트워크 계층 6

## Distance vector : link cost changes
- 한번 stabilize 된 이후에 link cost가 줄어 든 경우에는, 이 변화에 대한 새로운 결과를 구하고 빠른 속도로 안정화된다.
- 반대로 cost가 커진 경우, 큰 값으로 수정될 때까지 오랜 시간이 걸리고(`count to infinity`), 그 이유는 다른 루트를 거쳐 가는 최소 길이도 자기 자신을 중간에 다시 거치게 되어 의존성이 있을 수 있기 때문이다. 예를 들어, y->x의 cost가 커져 y->z->x를 선택한다 했을 때, 사실, y->z->x가 y->z->y->x였을 경우일 수도 있다. 이러면 의존성이 있어서 문제가 생긴다.
    - 이런 경우를 대비하기 위해 `reverse path를 막아야 한다`. dz(x)를 구할 때 `결정적인 역할을 한 neighbor`에게는 dz(x)를 구한 값이 아닌 `무한대로 해서 넘겨준다`.

## Hierarchical routing
- 인터넷은 엄청 크다. 여기서 routing algorithm을 돌리기는 힘들다. 그래서 `계층화` 시킨다.


### Interconnected ASes
- 각자의 네트워크는 네트워크 `내부에서는` routing algorithm을 돌린다.
- 네트워크 끼리의 routing은 또다른 routing algorithm을 돌려 연결한다.
    - 예를 들어, A에서 google로 접속을 원한다. 그러면 A가 속한 network 내부에서 gateway router까지의 알고리즘 하나, gateway router로 부터 google이 속한 gateway router까지의 알고리즘 하나 이런식으로 따로 돌아간다.
- 각 네트워크의 알고리즘은 각 네트워크 관리 주체가 스스로 정한다. 이렇게 `자치권`을 가진 하나의 시스템을 `autonomous system(AS)`이라한다.
    - 그리고, 이 AS 내부에서 routing을 정하는 알고리즘을 `Intra-AS` routing protocol이라 하고, AS끼리의 routing 알고리즘을 `Inter-AS` routing protocol이라 한다.
- 즉, 현대 인터넷은 routing도 계층화되어있다.

### routing protocol
- Intra-AS routing protocol
    - distance vector 구현한게 RIP
    - link state 구현한게 OSPF
    - 이 둘이 Intra-AS에 속한다. 네트워크 내부에서의 프로토콜이다.
    - 이들의 `목적은 최소 cost, 최단경로`다.
- Inter-AS routing protocol
    - BGP
    - `목적이 불분명`하고, 관리 주체도 불분명하다. 최단경로가 아닌 정치, 경제 논리도 개입된다.

### AS
- 하나의 자치권을 가진 routing domain
- 각 AS는 각자 자신의 `고유한 번호`를 부여받는다.

#### AS Numbers(ASNs)
- 16bit values
- 1번은 BBN이라는 기관이 차지했다. 미국의 군수업체로, 70년대 초반에 인터넷 프로토콜 제작을 수주받아 개발했다.
- 6만개 넘게 존재한다.

### Relationships Between Networks
- AS가 6만개라 해서 이들이 모두 동일하지는 않다. `서로 간의 위상이 다르다`.
- AS를 운영하기 위해서는 장비, 사람 등이 필요하다. 하지만 가장 중요한 것은 `돈`이다. 운영 주체는 자선 단체가 아니고, 어디선가 돈을 받아 서비스하고 다른 주체에 서비스를 제공한다.
    - 또, 예를 들어 `어느 AS는 다른 AS(예: 통신사)에 돈을 지불하고, 그들의 트래픽을 할당받아 운영`된다.

#### Customers and Providers
- AS 사이에서는 이런 제공자와 소비자의 관계가 형성된다.
- 인터넷 연결을 제공하는 AS가 `provider`이고, 사용자가 `customer`이다.
    - customer는 provider에게 돈을 지불하고 트래픽을 받아온다.
    - 이 관계는 계약으로 맺어진다. 누가 provider가 되는가? 상대는 우리 것을 필요로 하는데, 우리는 굳이 필요 없는 경우다. 또, 이 관계는 `상대적`이다.
- 근데, 둘이 비슷한 경우도 있다. 이 경우는 `peering 관계`다. 이 때는 서로 돈 안 교환하고 상부상조한다.
- 서로 `관계가 맺어진 네트워크끼리`는 넘나들 수 있다. 직접적인 peer나 provider-customer 간은 넘나들 수 있다.
    - 하지만 `peer와 peer 끼리는 넘나들 수 없다`. 해봤자 중간 측에서 이득이 없기 때문이다.

### BGP
- Border Gateway Protocol
- AS들의 border에 있는 router끼리의 routing protocol.
- `policy-based`. 최적화가 아닌 AS끼리의 정책에 따라 좌우된다.
- 예시
    - AT&T research라는 데서, 자신의 prefix를 홍보하고 있다.
    - prefix와 자기 자신의 AS Path 필드를 추가한다. 여기서는 6341이다.
    - 그 다음 7018을 거쳐 다른데로 간다. 그 때, AS Path는 7018, 6341로 추가된다.
    - 이런식으로 `hop을 지날 때마다 AS number가 추가되며 경로를 보내`준다.
    - 마지막에 받은 AS에서 경로를 결정할 때, 이렇게 온 여러 경로를 보고, 그 중 가장 돈이 덜 들거나 정책적으로 적절한 쪽을 `선택`한다. hops의 개수가 많아도 `내가 갑의 위치, 즉, provider의 위치에 있게 하는 경우가 많은 경우, 돈이 되는 경로`다.
- `customer로 보내기 > peer로 보내기 > provider로 보내기` 순으로 선호된다.