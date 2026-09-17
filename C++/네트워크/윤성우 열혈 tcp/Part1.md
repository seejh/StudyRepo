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
### 2-1 프로토콜 정의
멀리 떨어져 있는 두 호스트간에 데이터를 주고 받는 방식을 정하는 것.<br/>
프로토콜이란 컴퓨터 상호간의 대화에 필요한 통신 규약을 의미한다.<br/>
### 2-2 소켓 생성
시스템 내부적으로 리소스 할당(소켓) 후 파일 디스크립터를 리턴하는 함수
int socket(int domain, int type, int protocol);
domain: 사용할 프로토콜 체계 설정
type: 전송 타입 설정
protoco: 프로토콜 지정

* 프로토콜 체계 (Protocol Family)?
  * PF_INET
    * IPv4 인터넷 프로토콜
  * PF_INET6
    * IPv6 인터넷 프로토콜
  * PF_LOCAL
    * 한 PC 내부에서 프로세스 간 통신
  * PF_PACKET
    * 저수준 소켓, raw 소켓을 다룰 때 사용
    * raw 소켓: 프로토콜을 직접 만들어 사용할 수 있는 소켓
  * PF_IPX

* 소켓 타입 (데이터 전송 타입)
  * SOCK_STREAM
  * SOCK_DGRAM

* 프로토콜 선택

### 2-3 프로토콜 체계(Protocol Family)
### 2-4 소켓 타입
### 2-5 프로토콜 선택
### 2-6 윈도우 기반으로 구축하기



