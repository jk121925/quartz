### 일딴 구성요소 부터

우선 spring 프로젝트들은 기본적으로 계층화된 아키텍쳐를 택한다. 이와 유사하게 메시지 처리 시스템에도 보통에 사용되는 "파이프 필터 (pipes-and-filters)"모델을 따른다.

* 필터는 메시지를 생산하거나 소비하는 모든 구성요소이다.
* 파이프는 필터간에 메시지를 전송해준다.

### 그럼 Message는 뭐야?
Message는 자바의 Integer와 같은 객체와 메타데이터를 감싼 범용 wrapper를 이야기한다. 
따라서 Message는 Header와 Payload로 구분된다.
* Header는 ID, TimeStamp, correlation ID, 반환 주소같은 정보를 담는다.
* MQTT에서는 Header에 topic정보를 줄 수도 있다.
![[Pasted image 20231005155402.png|400]]

### Message Chennel
Message Chennel은 파이프 필터 아키텍쳐에서 말하는 "파이프"를 나타낸다. 프로듀서는 채널에 메시지를 전송하며, 컨슈머는 채널에서 메시지를 받아간다. 따라서 메시지 채널은 메시지를 처리하는 구성요소들을 분리해주며 메시지를 interception하거나 모니터링 할 수 있는 포인트를 만들어 준다.
![[Pasted image 20231005160000.png|600]]
* 메시지 채널은 point-to-point나 publish-subscribe protocol을 따를 수 있다.
* point-to-point 채널은 특정 채널로 전송한 모든 메시지는 딱 하나의 컨슈머만 받아 갈 수 있다.
* pub-sub채널은 각 메시지를 해당 채널의 모든 subscriber에서 broadcast한다.

#### Message Endpoint
~~나는 여기가 제일 이해가 안되더라 제기랄~~

Spring Integration의 주요 목표 중 하나는 inversion of control을 통해 solution 개발을 단순하게 만들어 준다는 것이다. 
>**즉 프로그래머는 컨슈머와 프로듀서를 직접 구현할 필요가 없으며 메시지를 만들거나 메시지 채널에서 send/receive 연산을 호출할 필요도 없다는 것이다**
>
>>대신 순순 객체를 기반으로 구현을 진행하고 가지고 있는 도메인 모델에 집중 할 수 있어야 한다.
>>이후 설정을 해주면서 도메인 관련 코드를 Spring Integration에서 제공하는 메시지 처리 인프라에 연결해 주면 된다 이 연결이 바로 Message Endpoint이다.

중요한 부분은 이상적으로 애플리케이션 코드에서 메시지 객체나 메시지 채널을 알지 못해야 한다는 뜻이다. controller가 HTTP 요청을 처리하듯 메시지 엔드포인트는 메시지를 처리한다는 것이다.
