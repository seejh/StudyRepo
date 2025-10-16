# IOCP
윈도우 전용의 고성능 비동기 소켓, 파일 입출력 처리 모델.
많은 클라이언트 연결을 효율적으로 처리하기 위해 커널 레벨에서 설계된 비동기 이벤트 큐 시스템이라고 생각하면 된다. <br/>

IOCP는 입출력 작업이 완료되었을 때 커널이 통보해주는 메커니즘을 가지고 있고 즉, 스레드가 I/O 작업(recv, send, accept 등)을 직접
기다리는 것이 아니라 I/O 요청을 걸어두고 완료되면 커널이 알려주는 비동기 이벤트 통지 방식. <br/>

게임 서버 등 고성능 서버가 필요한 경우에 많이 사용한다.

## 동작 흐름(대략)
CP 생성 -> CP에 등록 -> 비동기 I/O 요청 -> 완료 이벤트 대기 및 처리 -> 재등록
```c++
// 1. CP 생성
HANDLE iocp = CreateIoCompletionPort(INVALID_HANDLE_VALUE, NULL, 0, 0)

// 2. CP에 등록 (예:소켓)
SOCKET socket;
CreateIoCompletionPort((HANDLE)socket, iocp, ...)

// 3. 비동기 I/O 요청 (예:수신)
WSARecv(socket, ....)

// 4. 완료 이벤트 대기 및 처리
while (true) {
  bool success = GetQueuedCompletionStatus(iocp, &bytesTransferred, ...)
  if (success == false) {
    // 에러 처리
    continue;
  }

  // 재등록
  WSARecv()
}
```

## IOCP & Overlapped I/O
IOCP는 비동기 입출력 요청의 완료 통보 시스템이고 Overlapped I/O는 비동기 입출력 요청을 거는 방식.
이 두개가 합쳐져서 완전한 비동기 구조가 된다.

## 장점
OS(윈도우 커널)에서 비동기 처리, 스레드 풀링, 동기화 처리 등을 해주기에 메인 로직에 집중할 수 있고 
불필요한 컨텍스트 스위치를 줄여 고성능, 저지연 처리가 가능하고 
"Overlapped 요청 -> 커널 완료 -> IOCP 큐 통지 -> 워커 스레드 처리"라는 간단한 파이프라인이 성립.

## 단점, 유의점
page lock에 대한 문제(아래에 기재)

## 단계별 상세 흐름(커널 포함)
앱에서 Overlapped I/O 요청(예:WSARecv 등)을 걸면 윈도우 커널과 네트워크 드라이버가 실제 입출력을 수행하고 완료되면
커널이 Completion Port(완료 포트)에 완료 패킷을 넣어주고 앱은 GetQueuedCompletionStatus로 패킷을 받아 처리한다.<br/>

### 단계1: IOCP 생성
1. CreateIoCompletionPort()로 IOCP 핸들을 만든다.
2. 개별 소켓(또는 파일 핸들)을 CreateIoCompletionPort()로 IOCP에 등록한다.
3. WSAOVERLAPPED(또는 사용자 확장 구조체)를 준비하고 WSARecv 같은 Overlapped API로 비동기 요청

### 단계2: 커널로 요청 전달(네트워크 스택/드라이버에서 처리)
1. Winsock 레이어가 요청을 받아 커널 네트워크 스택(NDIS -> TCP/IP 등)으로 전달.
2. NIC 드라이버(또는 TCP/IP 스택)가 실제 패킷 송/수신 처리
3. I/O가 네트워크 계층/드라이버에서 실제로 완료되면(recv로 데이터가 들어오거나 send가 네트워크 버퍼에 복사 완료) 커널은 해당 I/O 요청의 완료 상태 결정(성공/실패/전송된 바이트 수)

### 단계3: 커널이 CP에 완료 패킷 삽입
1. I/O가 완료되면 커널은 I/O 완료 패킷(Completion Packet)을 만든다. 패킷에는 보통
   * ULONG_PTR CompletionKey
   * LPOVERLAPPED lpOverlapped
   * DWORD BytesTransferred
   * DWORD Error
   
   등이 포함된다.
2. 이 패킷을 Completion Port 큐에 넣는다.

### 단계4: 스레드 깨우기
1. IOCP는 커널이 몇 개의 워커 스레드를 동시에 깨울지를 제어한다. 이 정책은 IOCP를 만들 때 설정한 Concurrency 값과 현재 활동 중인 스레드 수에 따라 달라진다.
2. 커널은 큐에 패킷을 넣고 현재 처리 중인 스레드 수 < 허용 동시 처리 수면 대기 중인(=슬립) 워크 스레드 하나를 깨운다. 반대의 경우는 적재만 한다.
3. 깨운 스레드는 GetQueuedCompletionStatus가 반환되며 해당 패킷을 얻는다.

IOCP는 필요한 만큼만 스레드를 깨워서 CPU 과부하와 컨텍스트 스위치를 줄인다.

### 단계5: 앱의 GetQueuedCompletionStatus
1. 워커 스레드가 GetQueuedCompletionStatus()에서 반환
   * bytes = 전송된 바이트 수
   * key = completionKey(어떤 세션인지)
   * lpOverlapped = Overlapped 구조체 포인터(어떤 작업인지, Recv, Send, Accept)
2. 스레드는 key로 session 객체를 찾고, lpOverlapped로 어떤 작업이 끝났는지 구분.
3. 오류/종료 체크 -> 연결 종료 처리, 자원 해제 등 수행
   * 상대가 연결을 종료하면 recv 완료가 bytes == 0로 도착(0 바이트 전송)
   * GetQueuedCompletionStatus가 실패하면 GetLastError로 상세 오류 확인 가능
5. 필요한 후속 작업(Recv 재등록, 패킷 파싱 등)


<hr/><br/><br/>

# IOCP page lock 문제
## 페이지(Page)
운영체제는 프로세스의 메모리를 페이지(Page)라는 단위로 관리한다.
페이지는 보통 4KB(=4096bytes) 크기이다.

## 가상 메모리
프로세스가 사용하는 메모리는 실제 물리 메모리에 전부 올라가 있지 않아도 된다.
실제 메모리에 프로세스의 일부만 올라가서 작동한다는 것이고 운영체제는 필요할 때만 디스크(페이지 파일)에서
해당 페이지를 page-in 하고, 잘 안 쓰는 페이지는 물리 메모리에서 page-out 한다.
이를 가상 메모리(Virtual Memory) 시스템이라고 한다.

## 페이지 교체
프로세스가 현재 메모리에 없는 페이지에 접근하면 운영체제는 페이지 폴트(Page Fault) 인터럽트를 발생시킨다.
  * 디스크에서 해당 페이지를 읽어 RAM에 올림
  * 페이지 테이블 갱신
  * 다시 실행

디스크 I/O가 발생하기에 이 과정은 매우 느리다.(ms 단위, 메모리는 ns 단위)

## DMA(Direct Memory Access)
IOCP가 사용하는 비동기 I/O은 대부분 DMA를 사용한다. <br/>

* DMA

  CPU를 거치지 않고 디바이스(NIC, 디스크 컨트롤러)가 직접 메모리 버퍼에 데이터를 쓰거나 읽는 방식.
  예를 들어 WSARecv() 호출 시 커널은 사용자 버퍼 주소를 기록해두고 NIC가 DMA를 통해 그 주소로 데이터를 직접 써 넣는다.

CPU가 복사를 안 해도 되므로 매우 빠름. <br/>

DMA는 물리 주소로 작동한다. 윈도우는 가상 주소를 프로세스에게 보여준다. 여기서 발생할 수 있는 문제점이
  * 가상 주소는 언제든지 page-out 될 수 있다.
  * 만약 DMA가 진행 중인데 OS가 그 버퍼 페이지를 디스크로 내보내버리면?
    
그래서 운영체제는 DMA가 걸린 동안 그 페이지가 절대 스왑되지 않도록 페이지를 잠근다(Page Lock).

## 그래서 IOCP에서 Page Lock이 발생하는 이유
IOCP는 Overlapped I/O + DMA 기반으로 동작한다. 

```c++
//
WSABUF wsaBuf;
char data[1024];
wsaBuf.buf = data;
wsaBuf.len = sizeof(data);

// 
WSARecv(socket, &wsaBuf, ....)
```
이때 data는 유저 모드의 메모리 버퍼이다. 


<hr/><br/><br/>
# IOCP 외 다른 네트워크 모델
## Overlapped I/O
커널에 I/O 요청을 하고 커널로부터 완료 통지를 대기하는 방식으로 IOCP와 비슷하며 다만 차이점은 IOCP의 경우
커널로부터 완료 통지를 이벤트로 받지만 여기서는 WSAGetOverlappedResult를 호출하면서 폴링해야 한다.(busy )

## Select, WSAEventSelect
Select에서 블로킹 발생, WSAEventSelect는 해당 구간을 논블로킹, 이벤트로 통보 받는 방식으로 구현할 수 있다.
다만 이 방식의 문제점은 WSAWaitForMultipleEvents 함수가 받을 수 있는 최대 이벤트 개수의 한계가 정해져 있고 그 한계는
64개이다. 

