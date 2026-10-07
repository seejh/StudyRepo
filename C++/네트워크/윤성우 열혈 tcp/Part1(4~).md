# 4. TCP 기반 서버 클라이언트
### TCP / UDP에 대한 이해
TCP: Transmission Control Protocol의 약자로 전송 과정을 컨트롤한다라는 뜻
UDP: User Datagram Protocol

TCP/IP 프로토콜 스택<br/>
<img width="631" height="585" alt="image" src="https://github.com/user-attachments/assets/6537cbf3-d26f-460c-947a-dcbd885894ca" /><br/>
위의 그림과 같이 계층화가 되어 있다.<br/>

link 계층<br/>
물리 영역을 표준화하고 있다. 가장 기본이 되는 영역으로 LAN, WAN, MAN과 같은 네트워크 표준과 관련된 프로토콜을 정의하는 영역이다.
<img width="494" height="283" alt="image" src="https://github.com/user-attachments/assets/09573e90-1c7c-4d0b-b3a4-69e90c296935" />

ip 계층<br/>
link 계층을 통해 물리적으로 연결되었으니 여기서는 데이터를 보내는 것을 고려한다. 물리적으로 연결된 경로가 많고 이 경로들 중 어디로 보낼 것인지에
대한 선택을 (= 경로 결정) 하는 계층. ip 계층은 경로를 선택하는 계층이며 ip 프로토콜을 사용한다. ip 자체는 비연결, 신뢰할 수 없는 프로토콜이다. <br/>

tcp/udp 계층(전송 계층)
ip 계층을 통해 경로 결정이 되었으니 데이터를 보내면 된다. 방법에 따라 TCP, UDP로 나눠지는데 여기선 TCP에 대해서만 알아본다.<br/>

ip는 패킷 하나를 전송하는 것에만 중심을 두고 설계되었고 이 패킷이 정확하게 갔는지 패킷들이 순서에 맞게 갔는지는 고려하지 않는 프로토콜이다.
이를 신뢰성 있는 전송으로 만드려면 어떻게 해야 하는가? 

정리하자면 ip를 사용하며 데이터를 주고 받는데 거기에 룰을 정해서 안정성을 확보하는 것이 tcp이다.


### TCP 기반 서버 구현
### TCP 기반 클라 구현
### TCP 서버/클라 함수 호출 관계
### Iterative 서버 구현
### 에코(echo) 서버/클라 구현
### 윈도우즈 기반 구현
