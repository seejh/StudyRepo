# Part1 네트워크 프로그래밍 시작
## 1.네트워크 프로그래밍과 소켓의 이해
### 1-1 네트워크 프로그래밍의 이해
### 1-2 소켓 이해

### 1-3 파일 조작
유닉스 계열에서는 모든 것(콘솔, 소켓, 파일 등등)을 파일로 간주한다. 소켓과 파일은 유사한 점이 많기 때문에 파일을 이해하는 것이
소켓을 이해하는데 도움이 된다.<br/>

저수준 파일 입.출력(Low-Level File Access)<br/>
여기서 저수준이란 시스템이 직접 제공해 주는, ANSI 표준 C에서 정의된 함수들이 아니라 리눅스에서 제공해주는 함수라는 뜻.<br/>

파일 디스크립터(File Descriptor)<br/>
파일 디스크립터란 시스템으로부터 할당 받은 파일이나 소켓을 대표하는 정수를 의미한다. 윈도우의 핸들 개념. 파일 핸들이라고도 한다.<br/>

```c++
// 실행해보면 소켓이나 파일이나 똑같이 파일로 취급되어 생성 순서대로 디스크립터가 넘버링되고 있는 것을 확인할 수 있다.
// 결과 예:
// 파일 디스크립터1: 3
// 파일 디스크립터2: 4
// 파일 디스크립터3: 5
#include<stdio.h>
#include<fcntl.h>
#include<sys/socket.h>

int main() {
  int fdes1 = socket(PF_INET, SOCK_STREAM, 0); // 소켓 생성
  int fdes2 = open("test.dat", O_CREAT);       // 파일 생성
  int fdes3 = socket(PF_INET, SOCK_DGRAM, 0);  // 소켓 생성

  printf("파일 디스크립터1: %d\n", fdes1);
  printf("파일 디스크립터2: %d\n", fdes2);
  printf("파일 디스크립터3: %d\n", fdes3);
  close(fdes1); close(fdes2); close(fdes3);
}
```

### 1-4 윈도우 기반으로 구현
윈도우의 소켓(이하 윈속)은 유닉스 소켓을 참고하여 설계되어서 둘이 비슷하다. 윈도우에서 윈속을 사용하기 위해서는 winsock2.h 헤더를 포함해야 하고
해당 헤더를 포함시키기 위해서 WS2_32.lib 라이브러리를 링크해야 한다.<br/>

리눅스는 내부적으로 파일이나 소켓이나 모두 파일로 취급하기 때문에 파일을 생성하든, 소켓을 생성하든, 파일 디스크립터가 리턴된다. 
이전 예제(1-3 파일 조작)를 통해서 소켓이든 파일이든 동일한 넘버링으로 취급되는 것을 확인했다. 윈도우도 비슷하다. 
다만, 윈도우쪽은 이를 핸들이라고 부르며 소켓 핸들의 경우 SOCKET으로 표현한다.

```c++
// 서버
#include<winsock2.h>

int main() {
  // 윈속 초기화
  // 성공시 Wsastartup 함수 0 리턴, wsadata 변수에 로딩한 dll에 대한 정보가 채워진다.
  WSADATA wsaData;
  if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0)
    return 0;

  SOCKET listenSocket = socket(PF_INET, SOCK_STREAM, 0);

  // 리소스 해제
  WSACleanup()
}

```


## 2.소켓의 생성과 프로토콜의 설정
### 프로토콜
컴퓨터 상호간의 대화에 필요한 통신 규약을 의미한다.<br/>
멀리 떨어져 있는 두 컴퓨터간에 데이터를 주고 받는 방식을 정하는 것<br/>

### 소켓 생성
소켓 생성 함수
```
// 시스템 내부적으로 리소스 할당(소켓) 후 파일 디스크립터를 리턴하는 함수
int socket(int domain, int type, int protocol);


// int domain: 프로토콜 체계(Protocol Family) 설정
// int type: 전송 타입 설정
// int protocol: 프로토콜 지정
```

### 프로토콜 체계 (Protocol Family)
* PF_INET
  * ipv4 인터넷 프로토콜
* PF_INET6
  * ipv6 인터넷 프로토콜
* PF_LOCAL
  * local 통신을 위한 unix 프로토콜
  * local 통신=한 pc 내에서 프로세스간 통신
* PF_PACKET
  * Low level socket(raw 소켓)
* PF_IPX
  * IPX 노벨 프로토콜

### 전송 타입
* SOCK_STREAM (TCP)
  * 연결 지향형, 연결은 반드시 1:1
  * 데이터를 보내는 곳이 한 곳인데 받는 곳이 둘 이상이 될 수 없다.
  * 데이터 전달 보장(순서 변동, 에러, 손실 없이)
  * 전송되는 데이터의 경계가 없다

* SOCK_DGRAM (UDP)
  * 비연결, 빠른 속도 지향
  * 데이터 전달 보장 X (순서 변동, 에러, 손실 가능)
  * 전송되는 데이터의 경계 존재 (송신측에서 세 번 전송했으면 수신측에서 세 번 수신해야 한다.)
  * 한 번에 전송되는 데이터의 크기가 제한된다.

### 프로토콜
어떤 프로토콜을 사용할지 지정하는 곳으로, 앞에서 프로토콜 체계(Protocol Family)를 PF_INET으로 선택한다면,
여기서 사용할 수 있는 인자는 아래와 같다.<br/>

IPPROTO_TCP (TCP 기반 소켓=연결 지향)<br/>
IPPROTO_UDP (UDP 기반 소켓=비연결 지향)<br/>

근데, 프로토콜 체계를 PF_INET, 전송 타입을 SOCK_DGRAM이면 UDP잖아?<br/>
이렇게 앞에 두 개만 넣어도 결정되는 경우 이 세 번째 인자는 0을 넣어도 상관없다.<br/>
세 번째 인자는 RAW 소켓에서 유용하게 사용된다.<br/>

## 3. 주소 체계와 데이터 정렬
### IP 주소 (Internet Address)
인터넷상에 존재하는 호스트들을 구분하기 위한 32비트 주소 체계를 의미.<br/>
네트워크 주소와 호스트 주소로 이루어져 있다.<br/>



네트워크 주소란 네트워크를 구분해 주는 id를 의미.<br/>

### Port
한 PC 내에서 프로세스 구분<br/>

예를 들어 내 pc(ip 1.1.1.1)에서 인터넷 서핑을 하면서, 동영상을 보면서, 음악 스트리밍을 하면? 그러니까 ip 주소를 보고 호스트(pc)를 
찾아오는데 호스트 내부에서 구분은 어떻게? 이를 포트가 한다. 포트를 보고 필요한 프로세스로 전달한다. 이는 하드웨어의 물리적 할당이 아니라
소프트웨어의 논리적 할당이다.<br/>

tcp 소켓과 udp 소켓은 port

32비트 ip 주소로는 어느 네트워크인지, 해당 네트워크의 어떤 호스트(pc)인지를 구분하고 16비트 포트 정보로는 호스트 내에서
프로세스를 구분하게 된다. 이것은 물리적인 개념이 아니라 호스트 내에서의 논리적인 할당이다. 논리적인 할당이란? 하드웨어적으로 구현
되어 있는 것이 아니라 소프트웨어적으로 구현해 놓았다는 것. <br/>

tcp 소켓과 udp 소켓은 port를 서로 공유하지 않으므로 중복되어도 상관없다. 어느 tcp 소켓이 9190 port를 사용하고 있을 때
다른 tcp 소켓은 9190을 사용할 수 없지만 udp 소켓은 9190 port를 사용할 수 있다. 결론적으로 데이터 전송의 최종 목적지는
호스트(pc)가 아니라 호스트의 메모리상에 올라와 있는 실행 중에 있는 프로그램(프로세스)이다. 그러므로 데이터를 보내기 위해서
데이터 패킷 내에 ip 주소뿐만 아니라 port 정보도 있어야 한다.

### 3-3 주소 정보의 표현
ip 주소, 포트 등 주소 정보들을 표현해주는 데이터 타입(구조체로 선언되어 있는 데이터 타입)에 대해 알아본다. 모든 프로토콜은 자신만의
고유한 주소 포캣이 있다. 예를 들어 ipv4에서는 32비트 주소 체계를, ipv6에서는 128비트 주소 체계를 사용한다. ipv4 위주로 살펴본다.<br/>


* ipv4 주소 체계를 나타내는 구조체
```
struct sockaddr_in {
  sa_family_t        sin_family;  // 주소 체계 (address family)
  uint16_t           sin_port;    // 16비트 tcp or udp 포트
  struct in_addr     sin_addr;    // 32비트 ipv4 주소
  char               sin_zero[8]; // 사용되지 않음
};

* sin_family
프로토콜 체계마다 주소 체계가 다르다고 하였다. 여기에는 그 주소 체계에 대한 정보를 가지는 곳이다.
주소 체계 / 정의
AF_INET / IPv4 인터넷 프로토콜
AF_INET6 / IPv6 인터넷 프로토콜
AF_LOCAL / Local 통신(pc 내 프로세스간 통신)을 위한 UNIX 프로토콜

* sin_port
16비트 포트 정보
네트워크 바이트 순서대로 대입해야 한다.(네트워크 바이트 순서에 대해서는 잠시 후 언급)

* sin_addr
32비트 ip 주소 정보
네트워크 바이트 순서로 저장해야 한다.

* sin_zero
특별한 의미 없이 단순히 채워주기(padding) 목적으로 사용되는 구조체 멤버

struct in_addr {
  uint32_t s_addr; // 32비트 ipv4 인터넷 주소
};
```
이러한 데이터 타입들은 POSIX에서 그 근거를 찾을 수 있다. 
POSIX(Portable Operating System Interface)란 유닉스 계열의 운영 체제를 위해 표준화해 놓은 인터페이스(API)이다.<br/>

이제 위의 구조체의 의미를 이해할 수 있을 것이다. 근데 왜 이렇게 데이터 타입을 따로 선언해 놓은 걸까?
확장성을 고려한 결과라고 생각하면 된다. int32_t라는 데이터 타입을 사용한다면 어떠한 경우에도 4바이트 데이터 타입이라는 것을
보장받을 수 있다.<br/>


### 3-4 네트워크 바이트 순서
* 빅 엔디안(Big Endian)
  * 상위 바이트의 값이 먼저 표시되는 방법
* 리틀 엔디안(Little Endian)
  * 하위 바이트의 값이 먼저 표시되는 방법

* 네트워크 바이트 순서란? (Network Byte Order)
  * 네트워크 바이트 순서는 빅 엔디안으로 약속되어 있다.
  * 인텔 등의 일반적인 cpu는 리틀 엔디안을 사용한다.
  * 호스트(pc)에서 네트워크 전송할 때, 수신할 때 해당 바이트 순서로 맞춰줘야 한다.
  * 시스템이 리틀 엔디안인 경우
    * 송신할 때 빅 엔디안 변경
    * 수신할 때 리틀 엔디안으로 변경

* 바이트 순서 변환
```
h: host byte order
n: network byte order
s: short, 16bit, 포트
l: long, 32bit, ip 주소

unsigned short htons
unsigned short ntohs
unsigned long htonl
unsigned long ntohl

// 하나만 예를 들어서 표현 htons
// short(16비트) 데이터를 host 바이트 순서에서 network 바이트 순서로 변경 
```

또한 sockaddr_in 구조체 안의 모든 값들은 네트워크 바이트 순서로 채워져야 한다.
```
int main() {
  // 2바이트 데이터 (host 바이트 순서)
  short host_port_order = 0x1234; // 1234
  
  // 4바이트 데이터 (host 바이트 순서)
  long host_add_order = 0x12345678; // 12345678
  
  // 네트워크 바이트 순서로 변환
  // 시스템이 애초에 빅 엔디안이라면 아무런 일도 일어나지 않는다.
  short net_port_order = htons(host_port_order); // 12345678
  long net_add_order = htonl(host_add_order); // 78563412
}
```

### 3-5 인터넷 주소 조작
```c++
// Dotted-Decimal Notation을 Big-Endian 32비트 값으로 변환
// 성공 시 Big_Endian 32 비트 값, 오류 시 INADDR_NONE(-1) 리턴
unsigned long inet_addr(const char* string);

unsigned long addr;
addr = inet_addr("1.2.3.4");
addr = inet_addr("1.2.3.256"); // INADDR_NONE

// 성공 시 true(0이 아닌 값), 실패 시 false(0) 리턴
// 위랑 동일
// 
int inet_aton(const char* string, struct in_addr* addr);



// 성공 시 변환된 문자열의 포인터, 실패 시 -1 리턴
// 네트워크 바이트 순서 32비트를 Dotted-Decimal Notation으로 변환
char* inet_ntoa(struct in_addr addr);
```

### 3-6 인터넷 주소 초기화
### 3-7 주소 정보 할당하기
### 3-8 윈도우즈 기반으로 구현하기
### 3-9 WSAStringToAddress & WSAAddressToString












