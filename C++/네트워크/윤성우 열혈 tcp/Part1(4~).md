# 4. TCP 기반 서버 클라이언트
이 앞의 내용은 소켓의 생성과 소켓에 주소를 할당하는 정도로 이제부터 본격적으로 사용할 기반이 갖추어 졌다.

### TCP/UDP에 대한 이해
TCP: Transmission Control Protocol의 약자로 전송 과정을 컨트롤한다라는 뜻
UDP: User Datagram Protocol

TCP/IP 프로토콜 스택<br/>
계층화가 되어 있다.<br/>
<img width="631" height="585" alt="image" src="https://github.com/user-attachments/assets/6537cbf3-d26f-460c-947a-dcbd885894ca" /><br/>

link 계층<br/>
물리 영역을 표준화하고 있다. 가장 기본이 되는 영역으로 LAN, WAN, MAN과 같은 네트워크 표준과 관련된 프로토콜을 정의하는 영역이다.
<img width="494" height="283" alt="image" src="https://github.com/user-attachments/assets/09573e90-1c7c-4d0b-b3a4-69e90c296935" />

ip 계층<br/>
이제 물리적으로 연결되어 잇으니 데이터를 보낼 순서가 되었다. 복잡하게 연결되어 있는 인터넷을 통해 데이터를 보내기 위해 선행되어야 할 것은
경로를 선택하는 것이다. 이 문제를 해결하는 것이 ip 계층이며 이 계층에서 사용되는 프로토콜을 ip(Internet Protocol)라고 한다.<br/>

ip 자체는 비연결 지향이며 신뢰할 수 없는 프로토콜이다.

tcp/udp 계층(전송 계층)
데이터를 전송하기 위한 길 찾기를 ip 계층에서 해결했다. tcp/udp 계층은 데이터를 전송하는 방법을 정의하는 영역이다.
그 방법에 따라 tcp와 udp로 나눠지게 되는데 여기선 tcp

### TCP 기반 서버 구현
### TCP 기반 클라 구현
### TCP 서버/클라 함수 호출 관계
### Iterative 서버 구현
### 에코(echo) 서버/클라 구현
### 윈도우즈 기반 구현
