# 네트워크 계층 4
- IP 패킷은 header 부분과 data 부분으로 나뉜다.
- data에는 대부분의 경우 TCP segment가 들어간다.

## ICMP
- 사용자 데이터가 아닌 `네트워크 상의 증상을 알기 위한` control message를 보내기 위한 알고리즘.
    - 예를 들면, destination의 port가 열려 있지 않아 누락된 경우, TTL이 0이 되어 drop된 경우 등의 경우, source는 무슨 일이 일어난지 모른다. `이를 report해주기 위한 message가 필요하다`. 이는 네트워크 자체적으로 발생하는 control message이고, 이를 전달하기 위한 protocol이 `ICMP(Internet Control Message Protocol)`.
    - 예를 들어, TTL이 0이 되어 누락되면, data에 TTL 0이 되어 누락됨을 의미하는 정보가 담긴 IP packet이 생성되어 source로 전달된다.
- `네트워크 진단에 사용`된다.
- traceroute 명령에서는 특정 IP주소로 가는데 거치는 router를 알려준다. TTL이 1, 2, ...인 패킷을 보내며 TTL=0으로 drop되며 ICMP에 의해 report되는 정보를 통해 router 정보를 출력한다.

## IPv6
- address space가 각각 128bit(source, destination)
- 좀 더 단순한 header
- IPv4에서 IPv6로 가든 다른 곳으로 가든 과도기가 존재할 것이다. 이 동안 어떻게 하지?

### Tunneling
- IPv4, IPv6가 공존하기 위해서는 이에 대한 `변환`이 필요하다.
- 예를 들어 A~F까지 메시지를 보낸다. IPv6로 보낸 데이터의 헤더는 IPv4를 사용하는 곳에서는 해석이 불가능하다. 이 때는 새로운 형태의 패킷을 과거의 형태로 변환할 필요가 있다.
    - IPv6 header를 `data 부분으로` 보내고, IPv4 header를 `새로 생성`해서 보낸다. 이 때, `IPv4와 IPv6를 모두 이해하고 변환하는 router가 필요`하고, 여기서 tunneling 작업을 해준다.

## Routing Algorithm
- router는 forwarding, routing을 해준다.
- forwarding table을 보고 longest prefix matching 방식으로 forwarding을 해준다. 그런데, 이 forwarding table의 entry는 누가 채웠을까? `routing algorithm`을 이용했다.
- 네트워크를 `그래프로 개념화` 시키고 알고리즘을 적용하자.

### 그래프 추상화
- node = routers
- edge = links
- weight = cost
- 목적지까지의 최소 cost를 가지는 route 구하기(최소 경로 구하기).

### 두가지 접근 방식
1. 모든 라우터가 전체 그래프의 정보를 가지고, 경로를 구하는 경우.
    - link state algorithm
2. 라우터는 각자의 이웃과만 정보를 공유하며 구하는 경우.
    - distance vector algorithm

## Link State Algorithm
- 네트워크 전체의 정보 필요.
- 어떻게 알아야 할까? `모든 node들이 자기의 link 정보(link state)를 전체 network에 broadcast해줘야 한다`.
    - 그리고 난 뒤 무엇을 쓸까? `Dijkstra's algorithm`.

### 예시: router "u"의 forwarding table을 채워보자.
- 모든 `잠재적인 destination`(직, 간접적으로 연결된 모든 node)들을 forwarding table에 추가한다.
- notations
    - c(x,y) : x와 y 사이의 cost, 연결 안되면 무한대.
    - D(v) : source부터 v까지의 distance
    - N' : 현재까지 최단경로의 값이 알려진 node의 집합
    - p(v) : 여기로 가기 위해 직전에 거쳤던 노드.

- 일단, broadcasting을 통해 현재 그래프 정보를 모두 알고 있다고 가정하자.
1. 초기화
    - N' = {u} (자기 자신은 0으로 확실하므로)
    - 인접한 node의 D(node)값을 초기화(직접 연결되었을 때의 거리, 최소거리 아닐 수 있음)
2. loop
    - N'에 속하지 않으면서 distance값이 최소인 node를 찾는다.
    - 선택된 node가 w라고 하자, N'={u, w}가 되고, D(w)는 확정된다.
    - w와 인접하면서 N'에 속하지 않은 모든 node의 distance값을 업데이트 한다.
        - v의 값을 업데이트 한다면, `D(v) = min(D(v), D(w)+c(w,v))`. 즉, '기존값 vs w를 경유해서 가는 값' 중 최소값..
    - 모든 node가 N'에 속할 때까지 반복.
3. 이제 백트래킹
    - 직전에 거쳤던 node의 정보를 바탕으로 최단 거리로 갈 수 있는 route를 구한다.
    - forwarding table을 채운다. 위에서 구한 route 정보를 바탕으로 u에서 `다음에 갈` router의 위치를 적어준다.

#### 각자의 router들이 각자 알고리즘을 실행시켜 각자의 forwarding table을 완성한다.

### complexity
- n개 node의 경우, comparison 수행 때문에 O(n^2)이나, 효율적인 알고리즘을 쓰면 O(nlogn)이 된다.

### 논의(oscillations possible)
- 각자의 router가 각자의 forwarding table을 이용해 보내다 보니, link state algorithm을 실행시켰을 때, `그 때마다 network상황이 개입`해서 router 각자의 table에 영향을 끼쳐 네트워크가 여기로 몰렸다가, 저기로 몰렸다가가 반복된다.

### 현실적인 이야기
- 전 세계 모든 router의 map을 알 수는 없다.
- 그래서 여러 router로 이루어진 `하나의 네트워크`(`관리 주체가 동일한` 하나의 네트워크의 집합)를 정해서 그 범위에서만 routing algorithm을 수행한다.
- 이 내부에서만 broadcast를 수행해 네트워크 상황을 알아본다.

### 네트워크끼리의 routing은?
- 이것은 각자의 네트워크 관리 주체가 아닌 또다른 주체가 결정한다.

## Distance vector algorithm
- Bellman-Ford equation사용(dynamic programming).
- 나랑 직접적으로 연결된 친구들끼리만 메시지를 공유, 그 후, 전체 그래프를 유추해서 구한다.
    - 그러다보니 비직관적.

- dx(y) : x에서 y까지 가는데 드는 최소 경로의 비용
- dx(y) : `minv {c(x,v) + dv(y)}`
    - v는 x와 인접한 모든 노드의 집합
    - x에서 y로 가려면 무조건 `x의 이웃 중 하나를 거치게 된다`.
    - v = {a, b, c}라 하자. x -> y의 최소경로는 x->a->y, x->b->y, x->c->y 중 하나다.
    - 그러니 `이웃으로 가는데 드는 비용` + `해당 이웃으로부터 y까지 가는데 드는 최소 비용` 중 가장 작은 것이다.
    - 그리고, dv(y)도 `재귀방식`으로 구한다.
- 근데, 각자 dx(y)를 `구하면`, 이를 이웃으로 `넘겨주면서` 전체적인 최소 비용 route를 구할 수 있다.
