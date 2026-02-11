# future
비동기 작업의 결과를 나중에 받는 객체.

* 간단한 병렬 작업
* 스레드 직접 관리하기 싫을 때
* 반환값이 필요한 비동기 작업

주로 위와 같은 상황에서 비동기 작업의 결과를 나중에 받을 필요가 있을 때 사용.

## 기본 흐름
```c++
#include<iostream>
#include<future>
int work() {
  return 42;
}

int main() {
  /*----------------------------------------
    async() = 다른 스레드에서 실행
    future = 결과를 들고 있음
    get() = 결과 받을 때까지 block
  ----------------------------------------*/

  // 타스레드 하나 생성해서 work() 수행하고 결과를 future 객체 f에 넣어라.
  std::future<int> f = std::async(std::launch::async, work);

  // 다른 작업 가능

  // 여기서 결과 대기
  int result = f.get();
  cout << result << endl;
}
```
## 사용 예제
```c++
get()

wait()
결과 준비될 때까지 기다림
값은 꺼내지 않음
wait_for()


// 최대한 블로킹
auto status = f.wait_for(std::chrono::milliseconds(0));
if (status == std::future_status::ready)

// promise
int main() {
  std::promise<int> p;
  std::future<int> f = p.get_future();

  std::thread t([&p]() { p.set_value(100); });
  cout << g.get() << endl;

  t.join();
}

```
## launch 정책
std::launch::async
새 스레드 생성해서 수행

std::launch::deferred
get()이 호출될 때 수행

std::launch::default

