# 멀티미디어 네트워크
- 어떻게 유튜브같은 멀티미디어 네트워크가 동작하는가?
- application layer 이야기다.

## 멀티미디어란?

### 오디오
- 아날로그 신호를 디지털 데이터로 `변환`하는 작업이 필요하다.
- 아날로그 신호가 시간에 따라 amplitude가 `연속적으로 변한다`.
- `sampling`을 이용해 주기적으로 값을 저장한다.
    - 이 과정에서 오차가 발생한다.
    - 이 값을 나타내는 bit의 수가 클수록 오류가 줄어든다.
    - `더 촘촘이(더 자주 sampling), 더 많은 bit`를 쓰면 원본에 최대한 가까워진다.
- 결과값은 bps(bit per second)로 나오는데 초당 몇 bit의 디지털 신호로 변환되는가를 의미하며, 샘플링 주기와 각 샘플의 단계수로 결정된다. 이를 `coding rate`라 한다.
    - 예) 8,000 samples/sec, 256 quantized level(8bit) => 64,000bps
- sender가 이 coding rate로 전송할 때, receiver가 최소한 이정도를 받아서 처리할 수 있는 속도가 되어야 한다.

#### MP3
- 96, 128, 160kbps를 주로 사용한다.
- 96kbps는 1초간의 음을 96kb의 코드로 변환했다는 의미다.
- 더 높을수록 음질이 좋다.

#### CD
- 1.411 Mbps
- MP3보다 더 음질 좋겠죠?


### 비디오
- `이미지의 연속`이다. 예를들면 24 images/sec
- 각 이미지는 `frame`이다.
- 이미지는 `각 pixel의 색의 값으로 저장`하고, 일반적으로 이웃한 색은 비슷하거나 동일하니 이를 이용해 `압축이 가능`하다.
- `coding rate`로 품질을 나타낸다.
- sender가 이 coding rate로 전송할 때, receiver가 최소한 이정도를 받아서 처리할 수 있는 속도가 되어야 한다.


## 멀티미디어 주고받는 종류
- streaming : 저장된 영상, 음악을 보내준다.
- streaming live : 실제 영상, 음악을 실시간으로 보내준다. 저장된 것이 아니다.
- conversational : 통신에 사용되는 거
- 오늘은 streaming위주로 한다.

## streaming stored video
- 저장된 영화의 파일을 만든다.
- frame 순서대로 server가 client에게 보낸다.
- client는 이를 받아 이대로 띄워준다.

### 하지만 현실에서는...
- client가 받는 속도가 각 frame마다 다르다. network delay가 줄었다, 늘었다 하기 때문이다.
- 즉, 네트워크의 delay가 일정하지 않은 `network jitter`가 일어난다.

### 그래서...
- sender는 일정하게 보낸다.
- receiver는 다양한 delay로 된 정보를 받는다. 이대로 실행하면 안된다.
- 그래서 어느 정도 기다렸다가 제대로 처리해서 실행을 시킨다. 이를 `buffering`이라고 한다.
    - buffering이 길수록 끊기지 않는다(많이 받아 실행하니).
    - 하지만 사람 입장에서 delay가 심해지는 싫다.
- server는 동영상을 보내고, receiver의 buffer로 들어간다. 동영상 플레이어는 buffer에서 끄집어낸다.
    - buffer에는 `무언가 항상 채워져 있어야 해` 미리 다른 무언가를 채워놓는 경우도 있다.
    - 만약 buffer가 텅 비면? 갑자기 멈추겠죠? `버퍼링`. 이게 많으면 사용자들이 싫어하겠죠?
    - 멀티미디어 업체는 이러한 일을 피해야 한다.
- 예를 들어...
    - 유튜브 서버에서 브라우저에 영상 파일을 받아온다. 둘 다 application이다. 고로, `transport layer에 의존`한다.
    - 동영상이 2Mbps로 인코딩 되어 있다면? `이 이상의 속도로 보내야 한다`.
    - TCP vs UDP
        - UDP의 경우, 전송 속도를 내가 정한다. TCP는 네트워크가 정한다.
        - 어쨌든 2Mbps로 보내야 하니 `UDP가 더 유리할까`?
        - UDP면 현재 네트워크 상황을 고려하지 않는 것이다. 만약, 네트워크가 2kbps까지 가능하다면? 사용자는 쓰레기를 받을 것이다. 고로, `안된다`.
        - `TCP`로 보내면? 네트워크 상황 때문에 제대로 `전달 안된다`.

## DASH
- 그래서 유튜브는 `TCP`를 사용한다(네트워크 환경에 적응해야 하므로). 하지만 `DASH(Dynamic, Adaptive Streaming over HTTP)`라는 application technique를 사용한다.
    - http를 통해 보내고, 동적으로 네트워크 환경에 알맞게 한다.
- 유튜브로 매트릭스를 본다고 하자. 2시간 정도이니 2GB정도 되는 파일이다. 이를 통째로 인코딩하여 2Mbps로 보내는 것이 아니라, `작은 chunks로 나누어 보낸다`. Client는 각 chunks를 받아 순서대로 실행한다.
    - 한편, 이 chunks들은 하나의 coding rate가 아닌 `다양한 coding rate로 여러 버전을 만들어 놓는다`.
    - coding rate가 `높은 버전이 좋은 버전`이다. 반대로 coding rate가 `낮은 버전은 느린 속도로 전송해도 된다`.
    - 각 coding rate로 만들어진 chunk들의 url을 저장해 놓는다. 이를 mainfest file이라 한다.
    - youtube는 영상을 요청하면 영상이 아닌 각 영상의 manifest file을 보낸다. 여기에는 영상의 chunk와 각 chunk의 url table이 저장되어 있다.
    - client는 chunk 1을 가져올 때, 일단 작은 coding rate의 것을 받는다. 그리고 네트워크 속도를 보고 조금씩 coding rate가 높은 url을 요청한다. 그러다 속도 안나오면 낮은 coding rate의 url을 요청한다.
    - 유튜브 볼 때, 화질이 계속 변하는 이유는 이것 때문이다.
    화질 손해 봐도 연속적으로 보는 것에 만족한다.
- 유튜브라는 서비스는 사용자가 많다. 전세계 유튜브 사용자가 같은 시간에 `이 자료를 동시에 요청할 것`이다. `서버에 어떻게 저장`해야 할까?
    - 서버 한 곳에 놔두고 제공하면 `여기가 다운되면 모든 곳에서 문제가 생기고`, 트래픽이 여기로 몰리므로 해당 서비스 `근처 네트워크에 큰 delay`가 생기면서 `모두가 저화질`로 시청한다.

## 서버 문제 해결
1. multicast 기법(이건 개념으로만)
    - 지금까지는 sender와 receiver 사이에만 주고받는 `unicast 방식`이다. 하지만 여럿이 한 node로 몰리고 `같은 data`가 `여러 receiver`로 sending될 때, 문제가 발생한다.
    - 이를 해결하기 위해 server는 한번 보내준 뒤, 각자에게 알아서 분배하는 것이 `multicast`다. 근데, router가 이를 하는 것을 `구현하기 힘들다`. 그래서 대부분의 router는 지원 안된다(`그저 개념`).
2. Content Distribution Network(CDN)(현재 사용법)
    - application layer technique
    - 아얘 content가 저장된 storage를 `전세계 곳곳에 분산`시킨다.
    - 사용자가 요청하면, youtube는 manifest파일만 보내주고, client는 `가장 가까운 server`에서 url을 요청하며 영상을 받는다.
    - 이런 CDN 업체는 `infrastructure 업체`다.
    - 근데, manifest파일은 전세계 어디서나 동일하다. 어떻게 `같은 url을 이용하는데 다른 CDN 서버`로 연결되는가?
        - 예를 들어 업체가 어느 CDN업체랑 계약하면, client가 `해당 chunk 정보를 youtube 서버`로 요청하면, youtube는 `redirect를 통해 다른 url`을 알려준다. 이 url도 일단 세계에서나 동일하다.
        - 하지만, 이 url을 DNS를 통해 IP주소로 변환할 때, 일단 local name server에서 보고, 여기 없으면 `CDN 서버의 authorative의 DNS 서버`에서 해당 정보를 알려준다. 이 과정에서, 한국에서 물어본 client와 캐나다에서 물어본 client사이에 `다른 정보를 알려준다`. 이를 통해 작동한다.
        - DNS 쿼리는 결국 IP packet에 담긴 IP를 통해 어디서 왔는지 판별할 수 있다. 이를 바탕으로 다른 IP를 넘겨주면 그만이다.
    - 근데, NAT를 통해 다른 IP가 되지 않나?
        - 어차피 이것도 gateway router에 연결되고, sk broadband같은 `access network`를 거쳐 인터넷에 연결된다. `access network`는 인터넷 접속을 위해 거쳐가는 네트워크를 의미한다.
        - 이를 토대로 하면 된다.
    - 결국, CDN 서버는 `네트워크 상 hop수가 적은 곳`에 있는 곳에 있는게 좋다. 그렇다면 sk broadband같은 `access network 내부나 근처에 위치`시키면 빠르다.

## Netflix의 경우
- 유튜브보다도 고화질로 영상을 보내준다.
- 이곳도 amazon같은 `CDN 업체의 서버를 이용해 영상을 저장`하고, 원래 서버에는 `manifest 파일과 계정 관리 정도`만 한다.