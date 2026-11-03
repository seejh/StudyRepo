# Part3 윈도우 기반 프로그래밍

## 18. 윈도우 기반에서 스레드 사용하기

### 18 커널 오브젝트(Kernel Objects)
프로세스, 스레드, 파일, 세마포어, 뮤텍스...등 이러한 것들 이외에도 운영체제가 만들어 내는 리소스들은 참으로 다양하다. 스레드를 생성할 때는
CreateThread라는 함수를 사용하고 파일을 생성할 때는 CreateFilie 함수를 사용한다. 생성 방법은 다르더라도 이러한 것들은 커널이 생성해서 관리해주는
리소스들이다.<br/>

파일을 오픈해서 데이터를 읽고 쓰는 경우 커널은 그 파일에 대해 오픈 모드는 무엇이며 어디까지 데이터를 읽고 썼는지 위치 또한 기억하고 있어야 한다.
이러한 정보를 유지하고 갱신하는 것은 커널에 의해 이루어지며 이러한 것들이 관리이다. 커널에 의해 생성되는 리소스들을 관리하기 위해 커널은 내부적으로 데이터 블록을
지니게 된다. 커널 오브젝트란 커널에 의해 생성된 리소스를 관리하기 위한 데이터 블록을 말하는 것이고 커널의 소유이다.<br/>

### 스레드
윈도우에서 스레드는 프로그램의 실행 단위이다. 간단한 "Hello World!"를 출력하는 프로그램을 실행할 때, 윈도우는 일단 프로세스를 생성하고 그 후
프로세스 내부에 메인 스레드라는 것을 하나 생성한다. 이후 이 메인 스레드가 main 함수를 실행한다. 즉 프로그램의 시작점인 main을 실행하는 것은 프로세스가 아니라
프로세스 내부에 존재하는 스레드이다. 따라서 프로세스는 일을 처리하는 실행 단위가 아니라 스레드의 저장소라 볼 수 있다. 이러한 점은 유닉스 계열과의 차이점 중 하나이다.

```c++
DWORD WINAPI ThreadFunc(void* arg) {
  Sleep(3000);

  return 0;
}

int main() {
  HANDLE hThread;
  DWORD dwThreadId;

  hThread = (HANDLE)_beginthreadex(NULL, 0, ThreadFunc, NULL, 0, (unsigned*)&dwThreadId);
  if (hThread == 0)
    exit(1);

  printf("Thread Handle: %d\n", hThread);
  printf("Thread ID: %d\n", dwThreadId);
  Sleep(5000); 
}
```

### 커널 오브젝트와 핸들
<img width="540" height="236" alt="image" src="https://github.com/user-attachments/assets/bd16b3f0-79b4-45e3-b483-6a36eaa665f0" /><br/>
스레드를 생성하면 커널은 그 스레드를 관리하기 위해 커널 오브젝트를 생성하고 프로그램 영역에서 접근할 수 있게 핸들이라는 것을 발급한다.

### 커널 오브젝트 상태
커널 오브젝트에는 signaled, non-signaled 상태가 있다. 이를 위해 내부에 boolean 타입의 변수를 하나 가지고 있고 이 값이 false이면 non-signaled 상태이다.
이 상태가 무엇을 하는지는 각 커널 오브젝트 마다 다르다.

### WaitForSingleObject, WaitForMultipleObjects
커널 오브젝트가 signaled 되는 것을 기다리는 함수로 인자로 커널 오브젝트와 시간을 넘겨준다.<br/>
signaled가 되어 이 함수가 리턴되면 WAIT_OBJECT_0가 리턴.<br/>
timeout이 되어 이 함수가 리턴되면 WAIT_TIMEOUT이 리턴.<br/>
manual-reset 모드, auto-reset 모드란 것이 있는데 <br/>
auto-reset의 경우, signaled가 되어 함수가 리턴하면 자동으로 non-signaled로 바꿔주는 것이고<br/>
manual-reset의 경우, 수동으로 바꿔줘야 한다.<br/>

```c++
DWORD WaitForSingleObject {
  HANDLE  hHandle, // handle to object
  DWORD dwMilliseconds // time-out interval, INFINITE로 무한정 기다릴 수도 있다.
}

// 예제
// 스레드와 프로세스의 경우 종료되면 signaled 상태가 된다.
int main() {
  // 스레드 생성
  HANDLE hThread = (HANDLE)_beginthreadex(NULL, 0, ThreadFunc, NULL, 0, (unsigned*)&dwThreadId);

  // 스레드 종료 대기
  DWORD ret = WaitForSingleObject(hThread, INFINITE);
  if (ret == WAIT_FAILED)
    exit(1);
}
```

## 19. 스레드 동기화
하나의 프로세스 내에서 생성된 스레드들은 스택을 제외한 나머지 메모리 영역을 공유한다는 특징을 가지고 있다. 이 점에 의해 스레드간에 통신을 하기에는 편리하지만
스레드가 동시에 같은 메모리 영역을 접근할 경우에는 문제가 될 수 있다.

```c++
// 문제 발생 예제
```

### 커널 모드, 유저 모드
프로그램이 잘못된 연산을 통해서 운영체제를 손상시키거나 아니면 다른 프로세스의 메모리 영역 침범하게 되면 어떻게 될까? 이러한 경우를 막기 위한 개념이
커널 모드, 유저 모드이다. 프로세스는 실행 시 2가지 모드(유저 모드, 커널 모드)를 오가며 프로그램을 실행하게 된다.<br/> 

유저 모드:<br/>
하드 디스크와 같은 물리적 영역으로 직접적인 접근이 불가하며 접근할 수 있는 메모리 공간도 해당 프로세스가 할당 받은 공간만 가능하다. <br/>

커널 모드:<br/>

정리:<br/>
우리가 만든 프로그램은 기본적으로 유저 모드에서 실행되며 시스템 자원 접근이나 권한을 넘는 접근이 필요할 때 커널 모드로 전환되어 시스템에 의해 접근된다.

### 유저 모드 동기화, 커널 모드 동기화
유저 모드에서 동기화를 하는 경우에는 동기화 작업을 위해서 프로세스의 연산 모드를 변경할 필요가 없다.


## 20 비동기 Notification 입.출력 모델
윈도우에서 제공하는 확장된 입.출력 모델
### 20-1 비동기 Notification?
### 20-2 WSAEventSelect 모델 기반 서버 구현을 위한 API
### 20-3 WSAEventSelect 모델 기반의 에코 서버 구현




