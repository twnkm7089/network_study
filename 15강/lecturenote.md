# 링크 계층 1

## 개요
- 링크 계층에 대해 알아보자
- 유선 케이블 상황을 주로 다룰 것이다.
- 전송 단위 : frame
- 지금까지 상위 계층에서 한꺼풀씩 벗기면서 통신을 점점 구체적으로 알아보았다. 우리가 다른데로 패킷을 보낼 때는 일단 gateway router로 보내고, forwarding을 진행하며 점점 간다. 그런데 이것도 개념화된 것이다.
    - 실제로는 이 gateway router로 가는 길도 구체적으로 있다. host와 router간에 전용선이 있는 것처럼 보였지만, 실제로는 `여러 host와 router가 연결된 채널`이 있고, 파장은 매체(broadcasting media)를 따라 퍼져 연결된 모든 곳으로 `broadcast`된다.
    - 이런 상황에서 여러 host가 동시에 이야기하면? `collision`이 발생해 신호가 섞인다. gateway router에게는 신호가 섞인 결과, 이해 불가능한 잡음이 들린다. 이런 `충돌이 없어야` 다음 hop까지 메시지가 잘 전달된다.
- link layer의 역할은 이러한 `충돌을 피하거나 충돌에 대처하는 것이다`.

<hr>

- 결국 link layer의 이야기는 `한 hop을 어떻게 갈 것인가`의 이야기다. 즉, 한 hop에서 다음 hop으로 갈 때 충돌이 안 일어나고 잘 전달되게 하는 방법이다.
- 여기 쓰는 여러 기술이 있다. Ethernet, Wi-fi, ...등등. 이 모든 것은 `유선이든 무선이든 broadcast media에서의 collision 처리 방법이다`. 
- 우리는 일단 유선에 대해 다룰 것이다.

## 구현된 곳
- link layer는 `network interface card` 안에 구현되어 있다.
- NW layer에서 내려온 패킷이 network interface card로 들어간 후, 외부로 나갈 때 frame의 data로 들어가 전달되는 방식.

## Access Protocol
- host와 host 사이가 `전용선이면 문제가 없다`.
- 하지만 실제로는 전용선이 아닌 `여러 사람이 공유`하는 `채널, 링크를 사용`한다.
    - 그러면 한 쪽에서 signal을 보내면 다른 곳으로도 퍼지며 문제가 발생한다.
    - 어쨌든 한 쪽에서 보내면 다른 데로도 가니 `broadcast medium`이다.
    - 두명 이상이 이야기하면 신호가 섞인다(충돌).
- 매체에 접근할 때(Medium Access) 잘 조절(Control)해야 하고, 이 때 쓰는 기술을 `MAC`이라 할 것이다. 우리는 `MAC protocol`에 대해 배울 것이다. Wi-fi, LTE도 이러한 MAC protocol의 일종이다.

### 이상적 MAC protocol
- link의 채널 bandwidth가 R bit/second라고 하자. 노드는 1, 2, 3, 4가 있다고 하자.
- 하나의 노드는 R의 bandwidth를 모두 점유해야 한다.
- 여러 노드는 bandwidth를 균등히 나누어 점유한다.
- 분산 처리되고, 단순하면 좋겠다.

## MAC protocol의 3개의 카테고리
- channel partitioning
- random access
- taking turns

## Channel Partitioning : TDMA(Time Division Multiple Access)
- 각 사람별로 시간을 나누어 채널에 접속하도록 한다.
- 각 사람별로 `time slice`를 나눈 후, `자신의 차례일 때만 접속할 수 있도록 한다`.
- 충돌이 나지 않지만, `자원을 낭비한다`. 모두가 동시에 접속해 활발히 보내면 모를까, 아니면 망한다.

### Channel Partitioning : FDMA(Frequency Division Multiple Access)
- `주파수 단위`로 나눈 것이다.
- TDMA와 같은 장단점이 존재한다.

## Random Access Protocols
- 자신이 `원할 때 보내는 방식`이다.
- `충돌이 일어날 수 있다`. 그래서, 이 충돌의 탐지와 충돌 발생 후의 처리 방식이 중요하다.
- ALOHA, slotted ALOHA, CSMA, CSMA/CD, CSMA/CA
- Random Access Protocol의 단점 : 사람 많아지면 delay 커진다.
- 실제 가장 많이 쓰이고, `Ethernet은 CSMA/CD`를 사용한다.

### Random Access Protocols : CSMA(carrier sense multiple access)
- Carrier(매체)를 sense(감지)한다.
- `listen before transmit`.
    - broadcast medium에서는 어디서든 적용 가능.
    - 내가 보낼 메시지가 있어도 누군가 채널 사용 중이면 일단 멈추고 `기다렸다가` 누군가 채널을 `다 사용하면 사용하기 시작`한다.
    - 이 경우에도, `여럿이 동시에 기다렸다가 이야기를 시작하면 동시에 이야기하는 꼴`이 되어 충돌한다.

#### CSMA 충돌
- 1이 채널을 사용하려 한다. 1은 감지해보니 채널을 아무도 안 쓰고 있어서 메시지를 전송한다.
- 근데, 이 신호가 퍼지기 전에 4가 메시지를 보내려고 한다. 감지해서 없으니 메시지를 전송한다.
- 결과적으로 거의 동시에 여러 채널이 전송을 시작하며 충돌한다. `Propagation delay`로 인해 충돌이 생긴다.
    - Propagation delay를 없앨순 없다. 빛의 속도로 존재한다.
    - 현재, 충돌이 났는데도 서로가 동시에 계속 이야기한 상황이다. 이는 비효율적이다.

### CSMA/CD(collision detection)
- 충돌을 감지하면 `바로 전송을 멈춘다`.
- 그렇다면, 둘 중 `누가 먼저 다시 이야기`할 것인가? 중재자는 없고, 각자가 분산적으로 결정하는 상황이다.
- 충돌 감지 이후의 이야기
    - `binary(exponential) backoff`
    1. 우선 다 멈춘다.
    2. `연속적으로 m번의 충돌`이 있었다면, {0, 1, 2, ... , 2^m-1} 중 `하나의 숫자를 골라`, 해당 시간 단위만큼 기다린 후에 재전송한다. 충돌 횟수가 적을 때는 기다리는 시간은 짧지만 다시 충돌할 확률이 크다. 충돌의 횟수가 늘어날수록 선택하는 시간 범위가 늘어남에 따라 충돌 확률이 적어진다.
        - 이러는 이유는 또한, 몇 명이 충돌 상황을 겪는지를 모르기 때문이기도 하다. 일단 적은 경우부터 가정하고, 점점 수를 늘리는게 효율적이다.
- `backoff시간은 delay에 영향을 미친다`. 이는 `충돌이 많을 때 커지고`, 즉, `사람이 많을 때` delay가 일어난다.

## "Taking turns" MAC protocols
- 위 둘의 혼합형
- single point failure : 하나의 무언가에 의존하므로 거기서 문제가 생기면 전체 네트워크가 무너진다.

### polling
- 진행자 같은 `master controller`가 존재.
- `master`는 `slaves`에게 보낼 데이터 있는지 물어보며, 있으면 보내게 하는 방식으로 제어한다. 단점은 master가 죽으면 전체 네트워크가 죽는다.

### token passing
- central server같은 것은 없고, token이라는 special message가 있다.
- 이 token을 가진 host만이 데이터를 전송할 수 있다.
- 전송이 완료되면 다른 host에게 token을 전달한다.
- 단점은 누군가 token을 분실하면 전체 네트워크가 죽는다.