# Part3 윈도우 기반 프로그래밍

## 18 윈도우 기반의 스레드 사용
리눅스는 프로그램의 기본 실행 단위가 프로세스인 반면 윈도우는 프로그램의 기본 실행 단위가 스레드이다.<br/>

다중 접속 서버를 구현하기 위한 방법으로, 리눅스 기반에서는 멀티 프로세스 서버를 구현하기 위해 fork 함수를 사용할 수 있지만
윈도우에서는 이에 대응하는 함수 조차 존재하지 않는다.(물론 비슷한 기능의 함수는 있다.)<br/>
윈도우에서는 이를 스레드 기반으로 처리한다.<br/>

### 18-1 커널 오브젝트
프로세스, 스레드, 파일.... 그리고 세마포어, 뮤텍스 등 이러한 것들 외에도 운영체제가 만들어 내는 리소스들은 참으로 다양하다.
거의 대부분 우리가 만들어 달라 요구하는 것들이고 만드는 방법도 제 각각이다. 스레드를 생성할 때는 CreateThread 함수를 사용하고
파일을 생성할 때는 CreateFile 함수를 사용한다.<br/>

생성하는 방법이야 조금씩 다르다 하더라도 이러한 리소스들은 커널이 "생성"해서 "관리"해 주는 리소스들이라는 공통점이 있다.<br/>

예를 들어보자<br/>
파일을 오픈(open)해서 데이터를 읽고 쓰는 경우 커널은 그 파일에 대해 오픈 모드는 무엇이며 어디까지 읽고 썼는지 위치를 기억하고 있어야 한다.
프로세스를 생성하는 경우 모든 프로세스들은 자신의 고유한 id를 지닌다. 또한 이제부터 언급하는 스레드도 자신의 고유한 id를 지닌다. 이러한 여러 정보들을
유지하고 갱신하는 것은 커널에 의해 이루어지며 이러한 것들을 "관리"라 한다.<br/>

커널 오브젝트는 커널에 의해 생성된 리소스(스레드, 파일 등)를 관리하기 위해 생성되는 데이터 블록을 의미하고 이 커널 오브젝트의 소유자는 커널이다.
내 프로그램에서 커널 오브젝트를 생성, 사용하다가 프로그램을 종료한다고 해서 이 커널 오브젝트가 사라진다고 볼 수는 없는 것이며 
또한 커널 오브젝트 안에는 카운터(counter) 역할을 하는 변수가 존재한다. 이 카운터는 접근하는 프로세스의 수를 나타낸다.<br/>
이렇듯 커널 오브젝트는 커널에 의해서 생성되고 관리되는 것이며 스레드나 파일 등을 생성 및 사용하는 함수를 "명령"이 아닌 "요구, 부탁"으로 이해해야 한다.<br/>

### 18-2 윈도우 기반의 스레드 생성
윈도우는 프로그램의 실행 단위가 "스레드"이다. Hello World를 출력하는 프로그램을 실행한다고 할 때 윈도우는 프로세스를 생성 후 프로세스 내부에
메인 스레드라는 것을 생성한다. 이 메인 스레드가 main 함수를 실행한다. 즉, 프로그램의 시작점인 main을 실행하는 것이 프로세스가 아니라
프로세스 내부에 존재하는 스레드(메인 스레드)이며 프로세스는 프로그램을 실행시키는 일의 단위가 아니라 생성된 스레드를 담고 있는 저장소라 볼 수 있다.
이러한 점이 윈도우와 유닉스 계열의 차이점이다.<br/>

'''c++
// 이 예제는 메인 스레드가 끝나면 프로세스가 종료된다는 것을 알려준다.
// 그래서 메인 외에 별개의 스레드를 만들어 사용하는 경우 이러한 경우를 방지하기 위해서
// 메인 스레드에서 다른 스레드를 기다려주게 만들어야 한다는 거

// 스레드가 소멸되는 시점은 스레드에 의해 제일 처음 호출된 함수가 리턴할 때이다.
// 이러한 방법 외에 ExitThread 함수를 사용하는 방법도 있지만 리소스를 해제하는 과정을 반드시 거쳐야 하는 불편함이 있다.
// 따라서 일반적으로는 전자에 의한 방법을 더 선호하며 그 외에도 호환성 문제를 고려해보더라도 전자가 더 좋다.
#include<stdio.h>
#include<stdlib.h>
#include<process.h>

unsigned int WINAPI ThreadFunction(void* arg) {

	for (int i = 0; i < 5; i++) {
		Sleep(2000);
		cout << "Running Thread Func....." << endl;
	}

	return 0;
}

int main() {

	DWORD dwThreadID;

  // 스레드 생성 함수
  // 일반적으로 두 부분만 신경쓰고 나머지는 디폴트 값(0, NULL)로 채우면 된다.
  // 스레드에서 호출할 함수, 해당 함수에 전달할 인자
	HANDLE hThread = (HANDLE)_beginthreadex(NULL, 0, ThreadFunction, NULL, 0, (unsigned*)&dwThreadID);
	if (hThread == 0) {
		cout << "thread start error" << endl;
		return 0;
	}

	cout << "thread Handle: " << hThread << endl;
	cout << "thread id: " << dwThreadID << endl;
	Sleep(3000);
  
	cout << "Main End" << endl;
}
'''

정리하자면...<br/>
스레드는 커널에 의해 생성되고 관리되는 리소스이므로 스레드를 생성하는 함수를 호출하는 경우, 커널은 그 스레드를 관리하기 위한
커널 오브젝트를 생성한다. 그리고 프로그램상에서 그 커널 오브젝트를 구분하기 위해 핸들을 발급 받아 사용한다.<br/>
이를 도식화하면 아래와 같다.<br/>
<img width="658" height="317" alt="image" src="https://github.com/user-attachments/assets/4f8b145f-8ac8-44ab-bcea-d56fa10c3357" /><br/>
생성된 리소스(파일, 프로세스, 스레드 등)들에 대해 각각의 커널 오브젝트가 생성되어 있고 프로그램상에서는 핸들을 가지고 커널 오브젝트를 구분하고 있다.<br/>

### 18-3 signaled & Non-signaled 커널 오브젝트
위에서(18-2 코드) 메인 스레드가 끝나면 프로세스가 종료되는 것을 확인했고 이를 막으려면 메인 스레드의 실행을 지연시켜야 한다는 것이었다.
이 지연을 어떻게 시켜야할까?<br/>

커널 오브젝트의 상태<br/>
커널 오브젝트는 둘 중 하나의 상태를 지닌다(signaled, non-signaled). bool 타입의 변수로 표시되며 이 변수가 false이면 non-signaled.
이 상태의 의미는 커널 오브젝트마다 다르다. 프로세스와 스레드의 경우 실행중일 때 non-signaled이며 종료되면 signaled 상태가 된다.<br/>

WaitForSingleObject, WaitForMultipleObjects<br/>
```
#include<windows.h>

DWORD WaitForSingleObject(
  HANDLE hHandle, // 커널 오브젝트의 핸들
  DWORD dwMilliseconds // 타임 아웃, 단위 ms, INFINITE일 경우 계속 기다림
);
// 커널 오브젝트가 이벤트 발생으로 인해 signaled 상태가 되어 리턴하는 경우 = WAIT_OBJECT_0 리턴.
// 타임 아웃되어 리턴하는 경우 = WAIT_TIMEOUT 리턴
// 추가로 auto-reset 모드 커널 오브젝트, manual-reset 모드 커널 오브젝트라는 것이 있는데..
// auto-reset = 이벤트 발생으로 WAIT_OBJECT_0 리턴하면서 신호가 non-signaled로 변경
// manual-reset = non-signaled로 돌아가지 않는다 = 수동으로 변경해야 한다.

그래서 이를 사용해서 18-2의 코드를 다시 해본다면..(18-2 코드에서 waitforsingle...부분만 추가)

int main() {
	HANDLE hThread = (HANDLE)_beginthreadex(NULL, 0, ThreadFunction, NULL, 0, (unsigned*)&dwThreadID);
	if (hThread == 0) {}

  // 스레드 종료 시까지 main 함수 지연
  // hThread에 신호가 올 때까지 대기 -> 스레드가 종료 후 signaled 변경 -> 이 함수에서 WAIT_OBJECT_0 리턴
  DWORD dw = WaitForSingleObject(hThread, INFINITE);
  if (dw == WAIT_FAILED)
    cout << "Thread Wait Error" << endl;

  cout << "Main End" << endl;
}
```

### 18-4 멀티스레드 프로그래밍의 문제점
하나의 프로세스 내에서 생성된 스레드들은 스택을 제외한 나머지 메모리 영역을 공유한다. 이러한 부분이 스레드간의
통신에는 편리하지만 스레드들이 동시에 같은 메모리에 접근할 경우에는 문제가 생길 수 있다. 이러한 문제는 동기화로 처리한다.<br/>

위의 내용을 제대로 표현하자면, 스레드를 기반으로 프로그래밍을 하는 경우(멀티스레딩) 
임계 영역이(Critical Section, 여러 스레드가 접근하는 부분) 존재할 수 있고
이러한 경우 한 번에 하나의 스레드만 임계 영역에 접근하도록 스레드간 동기화가 필요하다<br/>

## 19 윈도우 기반의 스레드 동기화
유저 모드 동기화, 커널 모드 동기화
### 19-1 스레드 동기화 기법 분류
유저 모드(User Mode), 커널 모드(Kernel Mode)<br/>
이중 모드 연산(Dual-Mode Operation)이라는 말을 한다. 이 말은 프로세스가 실행 시 두 가지 모드로 실행될 수 있다는 말이다.
프로그램을 실행시키면 유저 모드와 커널 모드를 오가며 프로그램을 실행하게 된다.<br/>

이렇게 프로세스의 실행 모드를 구분해 놓은 이유는 안정성 때문이다. 프로그램 상에서 잘못된 연산을 통해 운영체제를 손상시키거나
공유되는 리소스들(커널 오브젝트도 공유되는 리소스에 포함된다.)이 손상되는 일들이 발생할 수 있다. 예를 들어, 내가 짠 프로그램이
운영체제의 메모리 영역에 접근해서 잘못된 연산을 하는 일이 발생한다면 큰일일 것이다.

유저 모드
접근할 수 있는 메모리 공간이 제한되어 있고 하드 디스크와 같은 물리적 영역으로의 직접적인 접근이 허용되지 않는다.
우리가 짠 프로그램은 유저 모드로 동작한다.

커널 모드
그러면 이상하다. 우리는 프로그램으로 파일 등에 접근도 하고 커널 오브젝트를(스레드) 만들기도 하면서 시스템 리소스에 접근했다.
이 경우는?
모드 전환이 일어나기 떄문.
프로그램이 실행되는 과정에서 공유되는 시스템 자원에 접근하는 경우, 프로그램 레벨에서 직접 접근하는 것이 아니라 커널 모드로
전환되어 시스템에 의해 접근된다. 커널 모드에서는 시스템의 모든 메모리 영역으로의 접근이 가능하다.

정리하자면 우리가 구현한 프로그램의 기본적인 실행 모드는 유저 모드로 실행되며 시스템 리소스에 접근이 필요할 때 안전을 위해
커널 모드로의 전환이 일어나 시스템에 의해 처리된 후 다시 유저 모드로 변환되어 프로그램을 실행한다.

유저 모드 동기화
윈도우에서 스레드를 동기화시키는 방법은 크게 유저 모드 기반, 커널 모드 기반이 있다.
유저 모드 동기화 기법으로는 CRITICAL_SECTION 오브젝트를 사용하는 방법이 있다.
유저 모드와 커널 모드 변환은 많은 연산 과정을 거쳐 이루어진다. cpu가 할 일이 많다는 것이며 이러한 것은 부하다.
유저 모드 동기화를 사용하면 이런 변환이 필요없어 부하를 줄일 수 있으나 커널 모드에 비해 제한된 기능만을 갖는다.
CRITICAL_SECTION

커널 모드 동기화
유저 모드 동기화가 가지는 제한 중 하나는 이러한 유저 모드는 단일 프로세스 내에서 작동하는 것으로 만약
여러 프로세스간에, 간 프로세스가 가지는 스레드간에 동기화가 필요한 경우 유저 모드는 사용할 수 없고 커널 모드로 처리해야 한다.
또한 유저 모드 동기화는 잘못 구현했을 경우 데드락에 빠지기 쉽다는 문제도 있다. 커널 모드 동기화는 데드락에 빠지지 않도록 타임 아웃을
설정해 줄 수 있다.
Event, Semaphore, Mutex

데드락

이 데드락이 가지는 큰 문제는 컴파일 오류 조차도 발생하지 않는다는 것이다.  커널 모드 동기화는 대기 상태가 길어지면
해당 대기 상태를 그냥 빠져 나올 수 있도록 타임 아웃을 설정할 수 있지만 이것이 완벽한 데드락의 해결책은 되지 않는다.
프로그래머가 신경 써서 데드락 상황을 예측하고 막아야 한다.

### 19-2 CRITICAL_SECTION
유저 모드 동기화 기법.<br/>

```c++
CRITICAL_SECTION cs;
int sum = 0;

unsigned int ThreadTask(void* arg) {
	for (int i = 0; i < 1'0000; i++) {
		EnterCriticalSection(&cs); // CS 획득
		++sum; // 임계 영역 진입
		LeaveCriticalSection(&cs); // CS 반납
	}
}

int main() {
	DWORD dwThreadId1, dwThreadId2;

  // CS 초기화
	InitializeCriticalSection(&cs);

	HANDLE hThread1 = (HANDLE)_beginthreadex(NULL, 0, ThreadTask, NULL, 0, (unsigned*)&dwThreadId1);
	HANDLE hThread2 = (HANDLE)_beginthreadex(NULL, 0, ThreadTask, NULL, 0, (unsigned*)&dwThreadId2);
	if (hThread1 == 0 || hThread2 == 0) return 0;

	if (WaitForSingleObject(hThread1, INFINITE) == WAIT_FAILED) return 0;
	if (WaitForSingleObject(hThread2, INFINITE) == WAIT_FAILED) return 0;

  // CS 소멸
  DeleteCriticalSection(&cs);

	cout << sum << endl;
}
```

### 19-3 Mutex(Mutual Exclusion)
커널 모드 동기화(Mutex 커널 오브젝트를 사용한)
뮤텍스도 크리티컬 섹션과 비슷한데 얘는 커널 오브젝트라는 것<br/>
```c++
// 골격만

HANDLE hMutex;

void ThreadTask() {
  WaitForSingleObject(hMutex, INFINITE); // 뮤텍스 획득
  ReleaseMutex(hMutex); // 뮤텍스 반납
}

int main() {
  // 뮤텍스 생성
  hMutex = CreateMutex(NULLL, FALSE, NULL);

  // 뮤텍스 소멸
  CloseHandle(hMutex);
}

```

### 19-4 Semaphore
커널 모드 동기화(세마포어 커널 오브젝트를 사용한)
```c++
17장 세마포 참조
```

### 19-5 Event
커널 모드 동기화(Event 커널 오브젝트를 사용한)<br/>
다른 동기화들과 다른 점은 Event 방식은 대기 상태에 있는 여러 스레드를 실행 가능 상태로
변경해 줄 수 있다는 것이다.<br/>

<img width="677" height="308" alt="image" src="https://github.com/user-attachments/assets/c2959a96-1cbe-4488-9d7d-3afbfa8df2ac" /><br/>
여러 스레드가 non-signaled 상태로 대기 중에 있고 어느 다른 스레드에서 Event를 주면(ResetEvent) non-signaled에서 signaled로 바뀌면서 대기 중이던 모든
스레드가 깨어난다.

```c++

/*
HANDLE CreateEvent(
  LPSECURITY_ATTRIBUTES lpEventAttributes,
  BOOL bManualReset,
  BOOL bInitialState,
  LPTSTR lpName
);
중요한 bManualReset, bInitialState만 설명한다. 나머지는 NULL로 처리.

bManualReset
false = auto-reset 모드 = signaled 상태가 되었을 때 자동으로 non-signaled로 돌아간다.
true = manual-reset 모드 = ResetEvent를 호출해야 signaled -> non-signaled로 변환

bInitialState
생성 시 상태 설정, true = signaled로 시작, false = non-signaled로 시작

추가적인 설명으로 매뉴얼 모드는 여러 스레드를 한 번에 깨울 때 유용하다.
(auto 모드는 하나만 깬다.)
*/

HANDLE hEvent;

void ThreadTask(int n) {
	// 신호 대기
	WaitForSingleObject(hEvent, INFINITE);
	cout << "thread-" << n << " received event and awake." << endl;
}

int main() {
	// Event 오브젝트 생성
	hEvent = CreateEvent(NULL, TRUE, FALSE, NULL);
	if (hEvent == NULL) return 0;

	thread t1(ThreadTask, 1);
	thread t2(ThreadTask, 2);

	// Event 오브젝트를 signaled 상태로 변경
	SetEvent(hEvent);

	t1.join();
	t2.join();
}
```

### 19-6 멀티스레드 기반 서버 구현
채팅 서버, 채팅 클라이언트<br/>

채팅 서버<br/>
```c++

```

채팅 클라이언트<br/>
```c++

```




