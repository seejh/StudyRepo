# future, promise, async, packaged_task
* 비동기 수행
* c++ 11 표준 라이브러리
* 비동기 수행 완료 여부, 스레드간 값 전달

## 1. async
* 비동기 수행
  * std::launch::async : 새 스레드 생성, 바로 비동기 수행
  * std::launch::deferred : 현재 스레드에서, get() 호출되는 시점에 수행
* std::future 리턴
* std::thread와 비슷하다.

```c++
using namespace std;
int AsyncTask(int n) { return n + 1; }

int main() {
  // 1. launch::async 옵션
  // 즉시 새 스레드 생성하여 비동기 수행
  future<int> ft1 = async(launch::async, AsyncTask, 1);


  // 2. launch::deferred 옵션
  // 새 스레드 생성없이 이 스레드에서, get()이 호출되는 시점에 수행
  future<int> ft2 = async(launch::deferred, AsyncTask, 1);

  // 메인 스레드 작업중, 비동기 작업 아직 수행 안되고 있음.

  // get() 호출, 이 스레드에서 작업
  ft2.get();

  // async()의 리턴을 받아주지 않으면 비동기로 수행되지 않고
  // 그냥 일률적으로 수행된다.
  // "future<int> ft = async();" 가 아니라
  // "async();"로만 하는 경우
}
```

## 2. future, promise
* 비동기 작업 완료 유무, 스레드간 값 전달
* future: 값 받기를 대기 (=Consumer)
* promise: 값 설정 (=Producer)

```c++

// future는 복사 방지, 람다 캡처에서 & 명시 필요
```

### shared_future
* future.get()은 딱 한번만 호출할 수 있다.
* get() 호출하면 future가 이동(move)된다.
* future의 내부 멤버 중 future_status는 동적할당.

멀티스레드 환경에서 여러 스레드에서 future에 get()을 할 필요가 있는 경우가 있다.
이러한 경우에 shared_future를 사용한다.

```c++
void runner(shared_furute<void> sft) {
  // 여러 곳에서 여러 번 get()이 호출.
  sft.get();
  cerr << "get()" << endl;
}

int main() {
  promise<void> pm;
  shared_future<void> sft = pm.get_future();

  thread t1(runner, sft);
  thread t2(runner, sft);
  thread t3(runner, sft);

  // cerr는 cout과는 다르게 버퍼를 사용하지 않기 때문에 터미널에 바로 출력된다라고 한다.
  cerr << "set value()" << endl;
  pm.set_value();

  t1.join();
  t2.join();
  t3.join();
}
```

## 3. packaged_task

```c++
int AsyncTask(int x) {
  return x + 1;
}

int main() {
  packaged_task<int(int)> pt(AsyncTask);

  future<int> ft = pt.get_future();

  thread t(move(pt), 2);

  cout << "result: " << ft.get() << endl;

  t.join();
}
```

참고 : <br/>
https://modoocode.com/284#google_vignette <br/>

