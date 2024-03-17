시작은 MQTT였다...

대학교에서 네트워크 수업을 들을 때 반드시 배우는게 http이다. 여기서 나는 http는 protocol이라고 배웠다. 

내가아는 protocol은 통신 규약이다. 통신규약은 이런거지 데이터를 여기저기 전송하는 프로그램을 짜다 보면 어떤식으로 데이터를 보내야하는지 고민이 된다. 내가 편하게 객체로 가지고 있다가 직렬화해서 보내도 되고 csv로 보내도 되고 json으로 보내고 그냥 문자열로 보내도 되고 이걸 전부 binary로 만들어서 보낸다.

하지만 중요한 건 내가 A로 보내면 상대로 A로 읽어야한다는 것이다.
예를 들어서 나는 json 직렬화 해서 {a : 1, b:2} 이렇게 보내는데 이걸 아무 생각없이 csv로 파싱하면 똥이되는 거다. 
요기서 서로 "나는 A로 보낼 테니까 너도 A로 읽어"하는게 protocol 통신규약이다.

자 그럼 MQTT가 통신 규약인 이유는 뭘까?
또 혈압이 오르지만 https://mqtt.org/ 첫페이지에 MQTT is an OASIS standard messaging protocol for the Internet of Things 라고 써있다.

하지만 이걸 이야기 하려고 하는 것이 아니라 그 구조를 좀 보고 싶어서 이야기를 꺼낸다. 이 구조에 대해서 조금 더 알아보자.
### 경량화된 protocol

HTTP,이건 참 많이들 안다. 이전에 기억이 새록새록 나는데 네트워크 수업을 들으면서 packet snipping을 하면서 잘모르는 http header를 바이너리로 읽고 과제 했던 기억이 난다. 그때 좀 바이너리에 대한 무서움을 깨기도 했는데. 여튼 중요한건 요게 아니라

![[MQTT - MQTT-1.png]]

HTTP의 패킷의 구조는 이렇게 생겼다. 여기는 한번에 전송가능한 데이터량이 \#KB ~ \#MB 까지 갈 수 있다. 웹페이지의 정보를 body에 담을 수 없기 때문에 쪼개서 보내는 이유도 이러한 이유 때문이다.

하지만 이러한 상황은 MQTT에 비하면 굉장히 나은 상황이다.

![[MQTT - MQTT-2.png]]

Http는 그림상으로 보면 header가 총 4x6 = 24byte를 쓰고 있는데 MQTT는 지금 필수적으로 존재하는 header는 2btye에 불과하다. 이친구는 \###byte ~ \#KB에 불과하다. 이러한 이유로 일딴은 매우 가벼운 protocol에 속한다.

그 이유는 간단하다. 예를 들어 로봇팔에 컴퓨터 만한 메모리와, cpu, 그래픽카드 이런것들을 달 수 있을까?
물론 달 수 있다. 하지만 투자대비 효용이 그렇게 나오지 않을 것이다. 단순한 명령을 처리하는 하드웨어에서 저렇게 많은 데이터를 보낼 일도 없을 뿐더러 저렇게 무거우면 예측가능한 시간안에 작동을 하지 않을 것이다. 해서 용량을 줄여 꼭 필요한 데이터만 payload에 담아서 보내는 것이다. 

### Messaging

TCP layer에서는 3-way handshake를 하고난 뒤에 데이터를 전송하게 된다. HTTP는 각 end-point가 데이터를 직접적으로 주고 받는다 (물론 라우터가 있고 그러지만)

하지만 MQTT는 Publish/Subscribe구조를 가지고 통신을 한다.

![[MQTT - MQTT-3.png]]

위에를 보면 publiser, broker, subscriber가 있는데 각역할은 다음과 같다.
1. publiser
	1. 데이터를 생성하는 주체다.
	2. 데이터를 ip:port로 게시한다.
	3. 여기서 게시란 broker에 message를 전송하는 것을 이야기한다.
	4. QoS(데이터의 전송 신뢰에 대한 flag)에 따라서 달라지기는 하지만 기본적으로 message를 보내고 나면 이에 대해서 더이상 신경쓰지 않는다. (http protocol은 message에 신뢰성에 대해서 신경을 많이 쓰지만 MQTT는 그것보다는 우선 게시하는 것이 더 중요하다.)
2. Broker
	1. 일종에 중계자로 이해하면 편하다.
	2. Clinet에서 데이터를 받아서 Topic에 저장한다. Topic은 publiser가 구독을 하는 key와 같은 역할을 하고 subscriber가 consume하는 key이기도 하다.
	3. 보통 고가용성(실패에 대비)을 가지는 여러개의 서버로 구성이 된다. MQTT에서는 HiveMQ가 대표적인 Broker이다.
3. Subscriber
	1. 구독자로 Broker에서 Topic을 기준으로 데이터를 polling할 수있다.
	2. 여러개의 구독자가 1개의 Topic에서 데이터를 가져올 수 있다. 1:N의 구조

그렇다면 왜 이런구조를 취하고 있을까? 하는 의문이 들었다. 여기서 내 나름의 생각은 아래와 같다.
	1. 언제 무슨일이 생길지 모른다. (안정성)
		1. 보통의 client - server 구조에서는 client의 요청에 server의 어떤 Thread는 응답을 보내도 clinet가 이를 기다리기도 한다. 
		2. 하지만 기기가 움직이는 도중에 이러한 통신이 끊기거나 중간에 오류가 발생해서 멈추게 된다면 치명적인 문제가 생길 수도 있을 것 같다. 특히 hardware에 들어가는 프로세서들은 컴퓨터 많큼 좋은 환경이 아니다. 메모리도 작고 cpu도 작으며 전력이 불안전할 경우 기기 작동에 신뢰성이 매우 떨어질 가능성이 있다.
		3. 이러한 부분을 막기 위해서 우선 모든 end-point를 분리하면서 분명히 각 end-point는 한가지의 책임만 가지고 있게 설계를 하지 않았나 하는 생각이 든다.
	2. 1:N의 구조 (HUB)
		1. broker는 자체적으로 cluster를 가질 수 있을 때 Cluster를 단일 구조로 친다면(해당 클러스터의 목표는 fail over에 대응하기 위함으로) N : 1 : N의 구조를 가질 수 있다.
		2. client - server 구조는 1:N의 구조를 가지고 하나의 Thread는 하나의 client와 통신한다. 하지만 생각해 보면 전세계에 인구의 M배로 많은 IoT기기들이 존재할 것이고 이 데이터를 관리하거나 시각각화 해서 보고 싶은 상황은 훨씬더 많은 것이다.
		3. 이러한 문제를 해결하기 위해 다양한 IoT기기에서 데이터를 모아서 다시 뿌려주는 구조를 만들지 않았나 하는 생각을 한다.
	3. 지속적 연결
		1. 네트워크를 배울 때 블루투스(이동형 통신)는 수시로 end-point가 바뀔 가능성이 높아 안정성이 상대적으로 떨어진다고 배웠다.
		2. IoT가 데이터를 broker와 연결을 시작하면 http처럼 연결을 끊지 않고 지속적으로 유지한다. 
		3. 연결을 지속적으로 유지한다는 면에서보면 리소스가 낭비될 수 있을 수 있지만 기기를 재연결하고 재가동하는 시간과 리스크 보다는 지속적으로 유지하면서 모니터링하는 면이 좀 더 유용할 수 있을 것이라 생각한다.

### QoS

이걸 볼때마다 Quality of Service라고 읽어야 하는지 아닌지를 고민하지만 QoS는 신뢰성에 관한 문제다. 

1. QoS == 0 에서는 publisher가 데이터를 보낼 때 broker에 제대로 갔던 갔지 않았던 신경쓰지 않는다. (At most one)
2. QoS == 1 에서는 publisher가 데이터를 보낼 때 최소 한번(At least one)을 확실하게 보낸다는 것이다. 이는 broker가 데이터를 받고 나서 다시 받았다는 메시지를 보내준다. 이때 QoS1이라는 flag가 들어간 packet에서는 broker에서 데이터를 받는 순간 puback이라는 message를 다시보내준다. 이를 확인할때 까지 다시 보내게 된다.
	![[MQTT - MQTT-4.png]]
3. QoS == 2에서 정확히 한번만 보낸다. 이게 뭐냐 Exactly Once인데, 아래순서로 보낸다.
	![[MQTT - MQTT-5.png]]
	1.  clinet가 데이터를 보낸다 
	2. 서버가 데이터를 받으면 PubRec packet을 client로 보낸다,
	3. 만약 이게 오지 않는다면 계속 데이터를 보낼 것이다.
	4. pubrec packet을 받은 client는 데이터가 우선 제대로 갔음에 안도하고 같은 메시지를 다시 보내지 않으며 pubrel packet을 보낸다.
	5. 해당 packet을 받은 server는 최초~ pubrel전까지 packet을 모두 제거하고 1개만 처리한다,
	6. 이후 다시 pubcomp를 보내게 된다.
	7. 마지막으로 pubcomp를 받은 client는 데이터에 pubcomp packet을 기록하고 이를 더이상 보내지 않는다.

마지막 QoS가 제일 복잡하지만 단 한번을 중복없이 무결하게 보낼 수 있다는 건 유의미 한것 이라고 생각한다. 다만 여러 Trade Off가 존재하는 것을 알아 둬야 한다.

1. QoS의 Level에 따라서 당연히 cpu overhead가 생긴다. QoS2에서는 많은 통신이 이루어져서 정확하게 통신가능하지만 긴급한 상황에서는 별 도움이 되지 않는다. 서버가 처리를 해야 clinet에서 사용을 할 수 있기 때문이다.
	1. 따라서 QoS2는 정말 중요한 명령이 들어가는 "IoT 동작 변화"와 같은 명령에 써야 할 것이다.
2. 이에 반해 QoS 0은 정말 긴급한 상황에 쓸 수 있을 것이다. 기계 고장을 알리는 역할을 QoS2로 하게 된다면 너무 늦게 대응을 할 수 있으므로 빠른 게시를 위해서는 QoS0정도를 쓸 수 있을 것이다.
3. QoS1정도는 중요한 로그를 남길때 사용할 수 있을 것 같다. 1정도면 데이터가 누락되는 일이 없을 것이기 때문이다.
하지만 역시 중요한 부분은 상황에 맞게 써야한다는 점이다. 기계가 로그를 남기는 것도 각기 다를 것이고 해당 로그에 현재상태를 보내는지 이동중인 상태를 보내는지 항시 필요한 데이터인지 특정 상황에 중요한 데이터인지를 파악해서 해당 Level을 정해서 써야한다.

### Conclusion

내가 맡은 파트는 결과적으로 전체 기획 및 backend의 구현이 었다. 하지만 결국 제일 공부를 많이 한 부분은 mqtt protocol이었다. 
한가지 깨달은 점은 상대방을 명확하게 설득하기 위해서는 내가 명확하게 알아야 한다는 것이다.
결과적으로는 대화하기 힘든 회로 개발자와 이야기를 후반에는 많이 하지 않았지만 초반에 내가 아는 지식과 이사람이 아는 지식을 합치 시키는 게 가장 어려웠다.

이해 해보려 해도 어떤 알력 싸움 같은게 있었던 것 같다. 나중에 같이 프로젝트를 하게 된 내가 아니꼬왔는지 계속 으름장을 놓았던 것이다. 내가 기획을 조금씩하면서 본인의 위치가 흔들려서 그랬다고 생각한다.

이사람을 설득시키는 방법은 보여주는 것이었다. 공부를 해서 실제로 구현을 하고 의도한대로 작동하는 모습을 보여주니까 설득이 되었다. 이건 이런 구조고 이렇게 해주면 이렇게 할 수 있다. 이런 이야기를 굉장히 많이 했던 것 같다. 그만큼 성장 했다고도 생각을 하기도 하고 다른 분야에 대해서 배워 볼 수 있었던 당시에는 몰랐지만 값진 경험이었다.


##### Network과제
packet snipping을 한 후에 해당 body를 출력하는 코드들이었다. 만점 받은 코드였는데 이때 3~4일 밤을 꼬박 샜던 기억이 있다. 2주 과제고 나오자마자 시작했는데 처음에 뭔지도 몰라서 헤멨던 기억이 있다.

다시 보니 엄청 열심히 했네 ㅎㅎ
```python
import pcapy
import os

# find all device that can use
devs = pcapy.findalldevs()
print(devs)

# choose device that we sniff
dev = input("Enter device name to sniff :")
print(dev)

# open live pcapy server
cap = pcapy.open_live(dev,65536,1,50)



# pcapy support getmask method for take netmask IPaddress 
# To use compile method for filtering we need int32 type's IP address
# this method can convert string IP address to int IP address 
# compile pcapy for filtering and set filter port 
def strIPaddr2intIpadd(strIP):
    listIP = strIP.split(".")
    intIP = int(listIP[0])*256*256*256 +int(listIP[1])*256*256+int(listIP[2])*256+int(listIP[3])
    return intIP

filter = "tcp and port 80"
pcapy.compile(cap.datalink(), 65663,filter,0, strIPaddr2intIpadd(cap.getmask()))
cap.setfilter(filter)



# check TCP has application layer about request
requestHeader = ["GET","POST","HEAD","PUT","DELETE","CONNECT","OPTIONS","TRACE","PATCH"]
def check_http_req(packets):
    tempStrPacket =""
    check_http_req = False
    check_packets_application_layer = False
    for packet in packets:
        tempStrPacket+=chr(packet)

    if(tempStrPacket.find("HTTP") !=-1):
        check_packets_application_layer = True

    for req in requestHeader:
        if (tempStrPacket.find(req) !=-1):
            check_http_req = True  

    return check_packets_application_layer,check_http_req
    


# application layer has http header and entity boby
# this method extract compact header data before enitity body 
def check_appLayer_end(packets):
    returnstr =""
    for packet in packets:
        returnstr+=chr(packet)
    end = returnstr.find('\r\n\r\n')
    returnstr = returnstr[:end+4]
    entity_data = packets[end+4:]
    return returnstr, entity_data



# getting Source IP and Distination IP byte code convert to decimal
def get_SD_addr(a):
    b = "%.2x%.2x%.2x%.2x" % (a[0],a[1],a[2],a[3])
    ip_return=""
    for i in range(0,len(b),2):
        ip = b[i:i+2]
        ip = str(ip)
        if(i==0):
            ip_return += str(int(ip,16))
        else:
            ip_return += ("." + str(int(ip,16)))
    return ip_return


def main():
    # check number of print packet data
    cnt =1
    # if packet has video data, check video packet until data fully received
    transfer_video = False
    
    

    while(True):
        ## GET NEXT PACKET
        # using pcapy make header and packet data
        # next method read next packet and return Pkther and packet
        (Pkther, packet) = cap.next()

        ## CHECK PACKET LENGTH AND HTTP HEADER
        # check ITotal length in IP header for extract http layer in TCP and http data
        # number [16:18] packet indicate Total Length except Ether header(14)
        # if total length is less than 54 means there are no optional TCP layer and no application layer
        # so we can extract packet that have application layer
        total_length = ("%.2x%.2x" % (packet[16],packet[17]))
        total_length = int(total_length,16)
        http_check, req_check = check_http_req(packet)

        print(packet)

        if(total_length >54):
            ## PRINT HTTP HEADER
            # http data is start to 54 byte
            # using check_appLayer_end method can extract http header line data except entity body
            # http_data is indicate application layer
            after_trasport = packet[54:]
            http_data_extract="" 
            entity_data=""
            
            print(packet)
            print()
            print(after_trasport)

            if(http_check == True):
                http_data_extract,entity_data=check_appLayer_end(after_trasport)
                # extracting Sort & Destination Port and IP
                # convert bytes to int and string 
                S_IP = get_SD_addr(packet[26:30])
                D_IP = get_SD_addr(packet[30:34])
                S_Port =("%.2x%.2x" % (packet[34],packet[35]))
                D_Port =("%.2x%.2x" % (packet[36],packet[37]))
                

                # port check
                # if the Source port == 80 means this packet is toward server for Request
                # if the Distination port ==80 means that packet is toward my computher, so this packet indicate Response 
                check_req_res = "Response" if ((int(S_Port,16) == 80) and req_check == False) else "Request" 

                # print number S_IP : S_port , D_IP : D_port and [request | Response]
                print(cnt, S_IP,":", int(S_Port,16), D_IP, ":",int(D_Port,16), check_req_res)

                # print http application layer data and check count that we print
                print(http_data_extract)        
                cnt+=1
            
            ## VIDEO RETRIEVE
            # if the application layer have Content-Type: video/mp4, that means this packet is the start of download mp4 data.
            if(http_data_extract.find("Content-Type: video/mp4") !=-1):
                # switch checker that packet with video raw data until the end of response
                transfer_video = True
                # open file and write byte datas start from ftyp (00 00 20 66 74 79 70) which indicate mp4 file
                # make directory if there isn't vid foler and write initial entity data
                os.makedirs("./vid",exist_ok=True)
                with open("./vid/file.mp4",'wb') as a:
                    a.write(entity_data) 
            # on this case after TCP layer there have mp4 data. So add the data what we generate.
            if(http_data_extract.find("Content-Type: video/mp4") ==-1 and transfer_video ==True and check_req_res == "Response"):
                with open("./vid/file.mp4",'ab') as a:
                    a.write(packet[54:]) 
            # if all request are collected by this function we request signal and trun off the function.
            elif(transfer_video ==True and check_req_res == "Request"):
                transfer_video = False
        
# call main 
if(__name__ == "__main__"):
    main()
```