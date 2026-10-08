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
ip 계층을 통해 경로 결정이 되었으니 데이터를 보내면 된다. 여기서는 데이터를 전송하는 방법을 결정하는 것으로 방법에 따라 TCP, UDP로 나눠지는데 
여기선 TCP에 대해서만 알아본다.<br/>

ip는 패킷 하나를 전송하는 것에만 중심을 두고 설계되었고 이 패킷이 정확하게 갔는지 패킷들이 순서에 맞게 갔는지는 고려하지 않는 프로토콜이다.
이를 신뢰성 있는 전송으로 만드려면 어떻게 해야 하는가? 이런식이다. A가 B에게 패킷을 보내면 B가 받았다는 응답을 A에게 하는 것. 만약에 A가 B에게
응답을 받지 못했다면 A는 B가 데이터를 수신하지 못한 것으로 간주하고 임의의 시간이 지난 다음 다시 전송을 한다. 이런 식으로 데이터를 주고 받을 때
확인 과정을 룰로 넣어서 데이터를 받는 것을 보장하는 것이 tcp이다. 정리하자면 ip를 사용하여 데이터를 주고 받는데 거기에 룰을 정해서 안정성을 확보하는 것이 tcp이다.
<br/>

application 계층<br/>
이전까지는 개발자인 우리 입장에서는 잘 확인하기 어려운 부분이었다면 여기서부터는 소켓이라는 도구를 사용해서 무언가를 만들어 내게 된다.

### TCP 기반 서버 구현
tcp 기반 서버 프로그램은 다음과 같은 수순을 지닌다.
socket(): 소켓 생성 ->
bind(): 소켓에 주소 할당 ->
listen(): 연결 요청 대기 ->
accept(): 연결 허용 ->
send(), recv(): 데이터 송수신 ->
close(): 연결 종료

int listen(int s, int backlog)
s: 소켓 파일 디스크립터
backlog: 연결 요청 대기 큐의 크기

연결 요청 대기 상태란?
서버가 "연결 요청 대기 상태"에 있다는 것 = 클라이언트가 연결 요청을 했고 서버가 그 요청을 받은 상태에서 아직 수락 안하고 대기하는 것.
그리고 대기는 대기실에서 하게 되는데 이 대기실을 backlog 큐.

int accept(int s, sockaddr* addr, int* addrlen)
s: 서버 소켓의 파일 디스크립터
addr: 클라이언트 주소
addrlen: 클라 주소 크기
```c++
#include<arpa/inet.h>
#include<sys/socket.h>

int main() {
  int listenSocket = socket(PF_INET, SOCK_STREAM, 0);
  if (listenSocket == -1)

  sockaddr_in serverAddr;
  memset(&serverAddr, 0, sizeof(serverAddr));
  serverAddr.sin_family = AF_INET;
  serverAddr.sin_addr.s_addr = htonl(INADDR_ANY);
  serverAddr.sin_port = htons(atoi(port));

  if (-1 == bind(listenSocket, (sockaddr*)&serverAddr, sizeof(serverAddr)))
  if (-1 == listen(listenSocket, 5))

  sockaddr_in clientAddr;
  int clientAddrLen = sizeof(clientAddr);
  int clientSocket = accept(listenSocket, (sockaddr*)&clientAddr, &clientAddrLen);
  if (clientSocket == -1)

  char msg[] = "Hello World\n";
  write(clientSocket, msg, sizeof(msg));

  close(clientSocket);
}
```

### TCP 기반 클라 구현
tcp 기반 클라는 다음 수순을 따른다.
socket(): 소켓 생성 ->
connect(): 연결 요청 -> 
read & write: 데이터 송수신 ->
close(): 연결 종료

서버가 listen 하고 있는 상태에서 클라가 connect()를 요청하면 서버의 backlog 큐로 들어가고 서버가 accept를 하면 연결이 완료된다.
int connect(int sockfd, sockaddr* serverAddr, int addrLen);
connect() 함수가 리턴되는 시점은 서버에 의해 연결 요청이 수락되거나, 오류가 발생해서 연결 요청이 중단되는 경우이다.
만약 연결 요청이 바로 이루어지지 않고 서버의 대기 큐에서 대기하고 있는 상태라면 connect 함수는 리턴되지 않고 블로킹 상태에 있게 된다.

추가로 클라의 경우 클라 주소 할당이 없었는데 상관없나? ㄴㄴ.
네트워크를 통해 데이터를 주고 받으려면 소켓이 필요하고
그 소켓은 ip 주소와 포트 정보가 있어야 한다.
connect 함수를 호출할 때 운영체제가 알아서 잡았다.
그러니까 connect 함수를 하면 운영체제가 알아서 소켓의 주소 정보를 채워줌으로 bind 함수를 명시적으로 호출할 필요는 없다.
```c++
int main() {
  sockaddr_in serverAddr;
  char msg[30];
  int clientSock = socket(PF_INET, SOCK_STREAM, 0);
  if (clientSock == -1)

  memset(&serverAddr, 0, sizeof(serverAddr));
  serverAddr.sin_family = AF_INET;
  serverAddr.sin_addr.s_addr = inet_addr();
  serverAddr.sin_port = htons(atoi());

  if (connect(clientSocket, (sockaddr*)&serverAddr, sizeof(serverAddr)))

  int readLen = read(clientSocket, msg, sizeof(msg) - 1);
  if (readLen == -1)

  msg[readLen] = 0;
  printf("%s\n", msg);

  close(clientSocket);
}

```

### 윈도우즈 기반 구현

# 5. TCP 기반 서버/클라이언트 2
TCP의 특징을 생각해서 만들어야 한다.
### TCP는 전송되는 데이터의 경계가 없다
전송되는 데이터의 경계가 없다는 말은 내가 보낸 패킷이 뭉쳐서 갈 수도 있고 반대로 쪼개져서 갈 수도 있다는 말이다.
예를 들어, 내가 데이터를 3번 보내면 상대방이 3번만 받아야할 것 같지만 실제로는 2번만에 다 받거나 보낸 3번보다 받는 횟수가 더 많을 수 있다.<br/>

왜 이런 일이 생기는 걸까?<br/>
데이터를 송수신할 때 우리 눈에는 우리가 만든 버퍼에서 상대방측으로 쏘는 것 같지만 사실은 커널이 송.수신 버퍼를 각각 만들어 둔다.
그래서 실제로 송.수신이 일어나면 아래와 같은 형태로 일어난다.
```
// a 송신 -> b 수신
a 유저 버퍼 -> a 커널 버퍼(송신) -> b 커널 버퍼(수신) -> b 유저 버퍼
```
a가 b에게 패킷을 전송하려 프로그램에서 send()를 하면 a의 커널 버퍼로 데이터가 복사되고 커널 버퍼에서 TCP에 의해 B의 커널 버퍼로
전송되게 된다. <br/>

수신측 버퍼가 40바이트인데 송신측에서 80바이트를 보내면 어떻게 되나요?<br/>
일단 TCP에서 이런 일은 일어나지 않는다. TCP는 슬라이딩 윈도우 프로토콜을 사용하여 흐름 제어를 사용하기 때문이다.
이게 뭔지 간단하게 설명하자면 송신측에서 수신측의 버퍼 여유분을 확인한 뒤 버퍼 여유분을 넘는 데이터는 전송하지 않는 것이다.

### TCP
TCP에 대한 설명은 밖에 따로 빼서 설명한다.

### TCP
TCP는 3단계를 거치게 된다. "연결 단계" -> "송수신" -> "연결 해제".<br/>

연결 단계<br/>
tcp는 송.수신을 하기 위해서 연결을 맺는 과정이 필요하며 이를 3way handshake라고 한다.
참고로 이 과정은 tcp 클라의 connect 함수 호출과 함께 시작한다.<br/>
<img width="376" height="361" alt="image" src="https://github.com/user-attachments/assets/501ac738-ef31-4213-ad87-429438b94ac9" /><br/>
SYN (Synchronization), 연결을 요청하는 패킷.<br/>
ACK (Acknowledgement), 상대방이 보낸 패킷의 수신 여부를 응답<br/>
SEQ (Sequence), 패킷 넘버<br/>

A->B (SYN): <br/>
a가 b에게 tcp 연결을 맺자고 보내는 패킷으로 내부에는 SEQ와 ACK가 있다. 현재 상황은 a가 b에게 연결을 요청하기 위해 SYN 패킷을 보내고 있으며
이 패킷은 1000번으로 지정하였고 이전에 받은 패킷은 당연히 없기에 ACK는 비어 있다.<br/>

B->A (SYN + ACK): <br/>
SYN 패킷을 받은 B는 A에게 "그래 연결하자"라는 뜻의 SYN 패킷을 날리는데 이 패킷에 이전에 A가 보낸 SYN 패킷에 대한 응답인 ACK도 포함된다.
B는 이 패킷에 2000번을 지정하고 이전에 A가 보낸 1000번 패킷을 잘 받았고 다음에는 1001로 보내면 된다라는 의미로 ACK를 1001로 지정해서 보낸다.<br/>

송.수신 단계<br/>
<img width="468" height="457" alt="image" src="https://github.com/user-attachments/assets/5a19fa8a-add6-452d-bae6-b94a33b897a6" /><br/>
A->B

### TCP
연결 단계 (3way handshake)<br/>
TCP는 송.수신을 하기 위해서 준비 단계를 가진다.<br/>
참고로 이 과정은 tcp 클라이언트의 connect 함수 호출과 동시에 시작한다.<br/>
<img width="376" height="361" alt="image" src="https://github.com/user-attachments/assets/501ac738-ef31-4213-ad87-429438b94ac9" /><br/>
A->B (SYN): <br/>
SYN, Synchronization의 줄임말로써 연결을 요청하는 패킷을 의미한다.<br/>
ACK, Acknowledgement의 줄임말로 상대방이 이전에 보낸 패킷의 수신 여부를 나타낸다.<br/>
이 패킷 안에는 SEQ와 ACK 정보가 들어 있고 SEQ는 현재 보내는 패킷에 넘버링을, ACK는 상대방이 보낸 패킷 수신 여부에 대한 응답이다.
현재 상황을 정리하면 A는 B에게 연결을 요청하기 위해 SYN 패킷을 보내고 있으며 이 패킷 번호를 1000으로 지정하였고 이전에 받은 패킷은 없기에 받은 패킷 응답인
ACK는 비어 있다.<br/>


B->A (SYN + ACK): <br/>
SYN 패킷을 받은 후 B도 A에게 "연결 준비가 되었다" 또는 "그래 연결하자"라는 뜻의 SYN 패킷을 날리는데 이 패킷은 이전에 A가 보낸 SYN 패킷에 대한 응답인 ACK도 포함된다.
B는 이 패킷에 패킷 번호 2000을 부여하고 이전에 A가 보낸 1000번 패킷을 잘 받았고 다음에는 1001로 보내면 된다라는 의미로 ACK를 1001로 지정해서 보낸다.<br/>

A->B(ACK): <br/>
B의 응답을 받았고 이제부터 통신을 개시하자라는 의미의 ACK를 보내게 된다. 1000번 다음 번호인 1001번으로 패킷 번호를 지정하고 "B가 보낸 2000번 패킷을 잘 받았고 2001부터
보내면 된다"라는 뜻으로 ACK를 2001로 지정해서 보낸다.<br/>

송.수신 단계<br/>
<img width="468" height="457" alt="image" src="https://github.com/user-attachments/assets/5a19fa8a-add6-452d-bae6-b94a33b897a6" /><br/>
A에서 B로 100바이트를 전송하면서 SEQ 1301을 전송한다. B에서는 응답을 받았다는 의미로 ACK 1401을 보낸다. 왜 이렇게 되는걸까? 1401번이 나오는 이유는
상대방 패킷 번호 1301 + 받은 바이트 100 = 1401이 되는 것이며 이렇게 바이트 수만큼 번호를 증가시켜 줘야 데이터 전송 유무만이 아니라 보낸 데이터를 다 받았는지
확인이 가능하다. 참고로 이 데이터라는 것은 페이로드에 대한 것이며 앞에서 TCP 핸드쉐이킹을 할 때 페이로드는 0이지만 1씩 증가하도록 약속되어 있다. 여튼 첫 번째
데이터 송수신은 성공했으며 두 번째는 문제가 생겨 B가 수신하지 못 했으며 이러한 경우 A는 일정 시간이 지나도 ACK 응답이 돌아오지 않으면 재전송을 시작한다.

연결 종료 단계<br/>
이 단계는 소켓 프로그래머에게 가장 중요한 단계에 해당이 된다.<br/>
<img width="379" height="430" alt="image" src="https://github.com/user-attachments/assets/5663ea3a-495c-43d8-85bc-3ecf25f29020" /><br/>
a가 b에게 종료 요청의 메시지를 담은 패킷을 전송한다. FIN
b가 a에게 끝내자는 FIN을 전송하는 것이 아니라 단순히 패킷을 잘 받았다는 ACK를 보내고 있다.
이것은 B가 아직 종료할 상황이 아니라는 것을 의미한다. 예를 들어 B는 아직도 전송해야 할 데이터가 남아 있는 상황을 생각해 볼 수 있다.
b가 a에게 자기도 종료 준비가 끝났다, 이만 끝내자는 FIN 패킷을 보낸다.
a가 b에게 최종 수신 ACK 응답을 보낸다.
이를 4way handshake라고 한다.

### 그래서 윈도우즈 기반에서 TCP 네트워킹을 구성할 때 어떻게?
저자분이 애매하게 끝냈는데 뒤에서 더 확인 해봐야 한다.

# 6. UDP 기반 서버/클라
tcp는 ip 기반의 비연결, 신뢰성 보장이 없는 통신에서 신뢰성과 연결성을 가지기 위해 여

tcp는 ip 기반의 비연결, 신뢰성 보장 없는 통신에서 신뢰성과 연결성을 가지기 위해 여러 노력을 해주는 방법이었고 udp의 경우 tcp가 해주던
흐름 제어를 하지 않는다는 것이다. 그래서 udp는 tcp에 비해 빠르다. 

udp 소켓이 전송하는 데이터의 기본 단위를 datagram이라고 한다.(일반적으로 udp 패킷이라는 용어를 더 많이 쓰기는 한다.)
이러한 데이터가 목적지 호스트로 길 찾아 가는데 까지 사용되는 프로토콜은 ip이다.

호스트 내에서 port 정보를 통해 최종 목적지인 프로세스를 구분하는 것. 이게 udp가 하는 일이고 이건 결국 tcp도 해주는 일이다.
몇몇 다른 일을 하기도 하지만 udp의 가장 중요한 작업으로 port 정보를 통해서 최종 목적지를 구분하여 주는 작업을 한다고 이해하고 있으면
충분하다.

TCP와 UDP 선택<br/>









