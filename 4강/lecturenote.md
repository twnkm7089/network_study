# 4주차

## 지난시간
- 전송 계층에는 TCP/UDP 존재
- 둘 다 공통적으로 해주는 것 : Multiplexing, Error Checking

## Reliable 통신의 원리
- TCP는 Reliable한 data transfer을 제공해준다. 어떻게?
- Reliable이란, Application에서 내려준 data가 유실 없이, 에러 없이 보내지는 것.
    - 하지만, 실제 라우터로 이루어진 네트워크 환경은 unreliable하다.
    - 이는 패킷의 유실과 패킷의 에러를 초래한다.

## 아주 간단한 Reliable Data Transfer protocol
- 한번에 패킷 하나씩 보내고, receiver가 받은 것을 확인하면 다음 것 보내기.

- 가정) underlying한 channel이 완벽히 reliable해서 패킷 에러, 패킷 유실 없으면?
    - 할일이 없다.
- 가정2) underlying channel이 패킷 에러가 발생 가능하다면?
    - `Error detection`이 필요.
        - 보내는 패킷에 checksum을 header에 붙여 실제 남긴 메시지에 패킷 에러가 있는지 판단. Receiver측에서는 그 패킷을 받아 관련 피드백을 남긴다. 만약 잘 받았으면 `ACK`, 못 받았으면 `NAK`을 줄 수 있을 것이다.
        - 만약 NAK이 돌아왔으면 `재전송(Retransmission)`한다.
    - 정리하면, Error가 있는 경우 필요한 것은 `Error detection, FeedBack(ACK/NAK), 재전송`.
- 가정3) FeedBack에 에러가 있다면?
    - ACK에 에러가 생겨서 잘 보내졌는지 판단 불가하면? -> FeedBack에도 checksum을 달아서 탐지 가능하다. 어쨌든 오류가 있다면? `다시 보내자`.
        - 근데 다시 보낸게 중복된 패킷인지 새 패킷인지는 어떻게 알지? receiver는 저번거 사실 잘 받았었다면 헷갈린다. 해결? `번호를 붙이자(sequence number)`.
        - Receiver는 sequence number보고 `중복된 패킷이면 버린다`.
    - sequence number는 `header`에 붙인다. 근데, 패킷이 많아지면 필요한 번호가 무한해진다. 이 정보를 최소화하기 위해서는 어떻게 해야 할까? 사실 지금의 경우 `2개(1bit)면 충분`하다.
        - 한번 보내고 상대방 받은거 확인하면 다음거 보내는 방식이니 0, 1, 0, 1 번갈아가며 보내면 충분하다.
        - 주의) sequence number가 같다고 같은 데이터가 아니다. seqence number는 어디까지나 구분용이다.

## NAK가 없는 프로토콜
- Receiver는 `제대로 받은 경우에만` 해당 `sequence number의 ACK`을 보낸다(예: ACK 0)
- 이제 `1번을 받아야 한다`. 만약 중간에 에러 나면? `ACK 0를 다시 보내면 sender에서는 NAK으로 인식`하고 다시 보낸다.

## 이제 유실이 있다.
- 메시지를 보냈는데 유실이 되었다. Sender는 아무것도 받을 수 없다.
    - `Timer`를 통해 해결한다. Sender는 패킷을 보낼 때마다 timer를 실행시킨다. 만약 유실되면 아무것도 안 돌아오니, timer가 time-out된다. 그러면 유실로 판단하고 재전송한다.
    - Timer의 시간은 얼마로 맞추어야 하나? '적당히'. 자세한건 TCP에서.
        - 왜 정해져 있지 않을까? Trade-off.
            - Timer가 짧으면? 유실에서의 recovery가 빠르다. 하지만, 중복된 패킷이 발생해 network에 overhead가 걸릴 수 있다.
            - Timer가 길면? 중복된 패킷은 적으나 유실에서의 recovery가 느리다.

## 지금의 프로토콜은...
- 보내고 확인까지의 시간이 너무 길다. 비효율적.
- 실제 TCP는 이런식으로 동작하지 않는다. 쫙 쏟아붓고(Pipelined), 각각에 관한 피드백을 받는 방식. 더욱 복잡해진다.