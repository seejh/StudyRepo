# 유저 모드, 커널 모드, 하이브리드
c++

## 유저 모드 동기화
커널의 개입 없이 사용자 애플리케이션 메모리 영역에서 원자적 연산을 통해 스레드를 동기화하는 방식.<br/>
컨텍스트 스위칭이 발생하지 않아 속도가 빠르지만 자원을 얻을 때까지 루프를 돌며 대기(Busy Waiting/Spin)하므로 cpu 점유율이 높아질 수 있다.

* std::atomic: cpu의 원자적 명령어를 직접 사용하여 락 없이(lockfree) 동기화
* spinlock: atomic을 사용해 직접 구현하며 락을 얻을 때까지 스핀
* interlocked 계열 함수 (windows): interlocked 함수를 사용하여 변수 단위의 원자적 연산 지원

## 커널 모드 동기화
커널이 직접 스레드의 상태를 관리하여 동기화하는 방식.<br/>
자원을 얻지 못한 스레드는 대기(sleep/blocked) 상태로 전환되어 cpu를 소모하지 않는다. 하지만 컨텍스트 스위칭이 발생해 상대적으로 느리다.
프로세스간 동기화(ipc)가 가능하다는 장점이 있다.

* Mutex (커널 뮤텍스): 프로세스간 동기화와 소유권 개념을 가진다.
* Semaphore: 지정된 개수만큼의 스레드 접근을 허용한다.
* Event: 특정 사건의 발생을 다른 스레드에 알릴 때 사용

## 하이브리드 - 현대 c++ 주류
유저 모드와 커널 모드의 단점을 보완하기 위해 처음에는 유저 모드에서 짧게 대기(spin)하다가 일정 시간 내에 락을 얻지 못하면 커널 모드로 전환해 
대기(sleep)하는 방식이다.

* std::mutex
* CRITICAL_SECTION (windows)
* condition_variable

## 요약 및 선택 기준
아주 짧은 시간 동안 단순 변수만 보호할 때: std::atomic (유저 모드)
코드 블록을 보호할 때: std::mutex (하이브리드)
서로 다른 프로세스간의 자원을 동기화할 때: Mutex(커널 뮤텍스) (커널 모드)

## 추가
유저 레벨 스레드(코루틴, Fiber, Boost.Fiber 등 커널 개입 없이 유저 영역에서 스레드 전환)에서 커널 모드 동기화 사용시 스레드가 잠들어(block) 버리면
실제 커널 스레드가 멈춰버려 전체가 멈춰버릴 수 있다.

처음 프로그램을 짤 때 명백한 단일 변수의 경우 std::atomic으로, 그 외는 std::mutex로 사용. 이후 프로파일링으로 최적화.
경합 줄이기: 락의 범위 쪼개기, 스레드별 로컬 데이터로 사용하다 한 번에 합치기 등으로 경합 자체를 줄이기
락 종류 변경: atomic 스핀락, 락프리

# 성능 프로파일링
코드 레벨, ide, os

## ide (비주얼 스튜디오)
* 성능 프로파일러: 기본
* 동시성 시각화 도우미(Concurrency Visualizer): 스레드 상태를 타임라인으로 시각화

위의 것들 사용할 때 빌드를 Release로 수행해야 한다.<br/>
동기화에 cpu가 낭비되는 정도 확인<br/>

### 성능 프로파일러
[프로파일러 실행] <br/>
상단 메뉴 -> 디버그 -> 성능 프로파일러 -> cpu 사용량 체크 -> 우측 톱니바퀴 클릭 -> 호출 수 수집 체크 -> 프로그램 시작

[분석] <br/>
현재 뷰: 호출 트리, 함수

* 외부 코드 해석
  * 외부 코드의 함수명이 ntdll.dll!0x7ff... 요런식으로 주소로만 보이면 microsoft의 윈도우즈 내부 심볼 파일을 다운로드해야 한다.
  * 상단 메뉴 -> 도구 -> 옵션 -> 디버깅 -> 기호 -> microsoft 기호 서버 체크

#### cpu 퍼센트 해석
* total cpu = self + 해당 함수에서 호출한 하위 함수에서 cpu 사용량
* self cpu = 해당 함수 자체 cpu 사용량

```
mutex g_m;
int g_count = 0;

void ThreadTask() {
	int count = 0;

	while (true) {

		g_m.lock();

		if (g_count >= 1000'0000) // 1억
			break;

		++count;
		++g_count;

		g_m.unlock();
	}
}

int main() {
	
	thread t1(ThreadTask);
	thread t2(ThreadTask);


	t1.join();
	t2.join();

	cout << "system end" << endl;
}
```

<img width="800" height="350" alt="image" src="https://github.com/user-attachments/assets/39448478-06ff-4733-ada8-6f21327f96a6" /><br/>
<img width="800" height="450" alt="image" src="https://github.com/user-attachments/assets/d228dd01-0242-4bc6-94aa-e32354db6c94" /><br/>
RtlAcquireSRWLockExclusive 락 획득 함수의 cpu 사용률 = 락 경쟁

### 동시성 시각화 도우미 (Concurrency Visualizer)
[확장 프로그램 설치 (최초 1회)]
상단 메뉴 -> 확장 -> 확장 관리 -> Concurrency Visualizer 검색 후 설치 (비주얼 스튜디오 재시작 필요)

[실행 및 분석]
상단 메뉴 -> 분석 -> Concurrency Visualizer -> Launch with Concurrency Visualizer<br/>
측정 후 스레드 탭 클릭<br/>
초록색(Execution): 정상적으로 cpu를 사용한 시간<br/>
빨간색(Synchronization): 락을 기다리며 멈춰 있는 시간<br/>


