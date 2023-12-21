# 네트워크 보안 1

## 네트워크 보안 요구조건
- confidentiality : 통신은 sender와 receiver사이의 메시지 교환이다. 이 메시지는 `제 3자가 알아서는 안된다`. `기밀성, 비밀성`. `암호화 기법`을 이용한다.
- authentication : `인증`. 내가 말하고 있는 상대가 정말 그 사람이어야 한다.
- message integrity : 보낸 사람의 메시지가 `그대로 가야` 한다. 중간에 `누가 변경하면 안된다`.
- access and availability : 서비스 제공자는 `24시간 사용자가 서비스를 사용 할 수 있도록` 해야 한다. `서비스 다운시키려는 상대를 막아`야 한다.
- 70년대에 TCP/IP 설계 당시에는 `이 개념이 없었다`. 그래서 그 후 공격이 생길 때마다 하나하나 추가된 상황이다.
    - 나중에 protocol이 생긴다면 이 개념들이 녹아들어야 한다.

## 용어
- Alice, Bob (lovers) : 착한사람들. 둘이 서로 대화를 하고자 한다.
- Trudy (intruder) : 나쁜사람. 메시지를 훔치거나 변형시키고 싶어한다.
- 우리의 목표는 Alice와 Bob이 Trudy의 방해를 뚫고 제대로 대화하도록 해야 한다.
- wiresharker를 키면 통신 상태가 다 보인다. 모든 사람이 Trudy가 될 수 있다. 모든 `네트워크 상황이 노출`되어 있다. `모니터링` 가능하다.
    - IP packet 다 보이니 header에 있는 `source, destination address를 통해 어디로 접속`하는지 다 보인다.
    - 어떻게 하면 이걸 모르게 할 수 있을까? `TOR`(The Onion Routing)을 이용해 어느 서버로 접속하는지 모르게 할 수 있다. TOR에 속한 `릴레이 노드가 전 세계 곳곳에 산재`해 직접 서버로 가지 않고 `이 노드들을 3곳 거쳐서 가게해서 최종목적지를 숨길 수 있다`. 모니터링 회피 가능하다.
        - 전세계에서 랜덤 3군데니 `느리다`.
    - data부분에 있는 TCP segment를 통해 TCP data영역에 있는 `메시지도 다 보인다` (`https`를 쓰면 그나마 여기는 막을 수 있다.).
- `warning.or.kr 사이트`.
    - 어떻게 http를 사용해 온 정보 중간에 warning.or.kr로 리다이렉트가 맺어질까? `TCP connection`을 맺을 것이고, `warning.or.kr이랑 connection을 맺은 적은 없는데`?
    - 내가 접근하려는 사이트를 어디서 차단한 것일까? 국가에서 차단하려는 사이트의 블랙리스트를 만든 후, `국외로 나가는 router에 이 리스트를 씌운다`. 그 후, 여기 접근하려는 유저에게 `경고장`을 띄운다.
    - 이를 우회하기 위해 해외에 존재하는 `proxy server`를 이용한다. `우리나라 -> proxy -> 서버` 같은 방식으로 `우회`할 수 있다.
    - 물론, 이 `proxy server까지 블랙리스트에 올리면` 막을 수 있다. 한국은 안 이러고, `중국, 북한`이 이러고 있다.
    - 그렇다면  warning.or.kr에 있는 warning message는 어떻게 날아온 것일까? 나는 `TCP connection을 맺은 적이 없는데`.
    - 우리가 도메인을 입력하면 `DNS server에서 IP 주소`를 받아온다. 그 후, TCP connection을 맺어야 한다. SYN segment를 보낸다. 여기에는 `sender, receiver의 주소`가 있다.
    - 이 과정에서 `블랙리스트가 있는 라우터`를 거친다. 이 과정에서 주소를 보고, 블랙리스트에 있는 목적지면 3-way handshake까지 허용해 `TCP연결은 일단 시키고`, http request의 `host field`에 블랙리스트에 있는 내용이 있으면 `warning.or.kr이 담긴 response message`를 만들고, `destination address에서 온 것으로 위장`한 후 보낸다. 
        - `그냥 drop 시키면 그냥 네트워크 오류`로 취급할 수 있다.
    - 고로, 우리나라는 IP주소가 아니라 `message의 host field`를 보고 검열한다. `Application layer`의 `firewall`.

## 암호화
- 메시지의 confidentiality을 보장하려면 평문을 `암호화` 시켜 주어야 하고, `key를 필요`로 한다. A가 보내는 `plaintext`가 `ka`라는 key로 암호화 되면 `ciphertext`가 되고, B가 이것을 받아 여기에 key `kb`를 적용하면 `plaintext` 복원된다.

### Symmetric key
- 암호화, 복호화에 `같은 키`를 사용한다.
- 기계적인 연산이라 `빠르고 효율적`이다. 하지만 같은 키를 공유해야 하므로, alice와 bob이 `사전에 만나거나 비밀 채널로 이를 주고받았어야` 한다.
- 70년대 초에 Diffie라는 사람이 이를 해결하기 위해 `Public Key Cryptography`를 만들어낸다.

### Public Key Cryptography
- 둘은 각자 공개키(`public key`)와 비밀키(`private key`)를 가진다. 공개키는 모두에게 공개되어 있다. A가 B에게 정보를 보낼 때, `A는 B의 공개키로 암호화` 시킨다. `비밀키는 B만 알고 있으니 이를 이용해 복호화`시킨다.
- 이를 실제로 구현한 것이 `RSA 알고리즘`이다.
- 메시지에 public -> private key순으로 적용하든, 반대로 적용하든 `결과는 같다`.
- 단점은 수학적으로 푸는데 시간이 `오래 걸린다`.

### 그래서 실제로는?
- `메시지 공유는 symmetric key`를 사용하고, `이 key를 전달하는 과정에 RSA 알고리즘`을 사용한다.
- 이를 통해 효율성과 기밀성을 모두 잡았다.

## Authentication
- 내가 이야기하고 있는 사람이 정말 이야기하고자 하는 사람인가?
- Alice가 Bob에게 내가 Alice임을 증명하기 위해 Bob은 Alice에게 `random number`를 보내고, Alice는 이를 둘이 서로 공유하는 `비밀키를 이용해 암호화` 시켜 보낸다. 그 후, Bob이 이를 복원해 `보낸 숫자가 나오면 된다`. 즉, Challenge를 주고 풀게 한다.
- 이 과정에서 `public key`가 사용된다. Bob은 Alice에게 R을 준다. Alice는 `private key`를 이용해 이를 암호화시킨다. Bob은 A의 `public key`를 이용해 이를 복원한다.

## Message Integrity
- 메시지가 중간에 변경되지 않고 보낸 것 그대로 보내지는가?
- `디지털 서명`
    - public key를 이용한다.
    - bob은 메시지를 `private key로 암호화`한다(`digital sign`).
    - alice는 bob의 `public key를 이용해 이를 복원`한다. 
    - 메시지를 받고 확인하는 과정이니, alice는 원본 메시지를 따로 알고 있는 상황이다. 복원 후, 원래 메시지가 나왔을 때, 이는 곧 `bob의 private key로 암호화되었었다는 것을 의미`한다. 즉, bob이 보낸 것이고, `중간에 메시지가 변경되지 않았음`을 의미한다.
    - 실제로는 너무 기니 `메시지의 해시값(message digests)`을 이용해 이 작업을 수행한다.

## Public Key의 문제점
- 근데, 우리가 bob의 public key라 알고 있는 것이 `정말 bob의 public key인지 확인`해야 한다. 이를 어떻게 확인해야 하는가?

## Public key 인증서
- 믿을만한 공인기관이 발행한 인증서를 들고다닌다.
- 이 인증서에는 `bob의 정보와 public key가 적혀 있다`. 이는 `인증기관의 private key로 암호화`되어있다.
- 우리는 인증기관의 public key로 이를 풀어 bob의 key가 맞는지 확인한다.
- 근데, 그러면 `인증기관의 public key가 진짜인지는 어떻게` 알 수 있는가? 그래서 이런 인증기관의 key들은 `우리의 browser 안에 내장되어 hard coding`되어 있다.
- 이런 인증서는 원래 `서버에서 들고 다녀야`한다. 그런데, 우리나라는 `이상하게 이걸 개인이 들고 다닌다`.