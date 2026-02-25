# future, promise, async, packaged_task
* 간단한 비동기 수행
* c++ 11 표준 라이브러리

## 1. future, promise
```c++
void worker(promise<string>* p) {
  p->set_value("some data");
}

int main() {
  promise<string> p;
  future<string> ft = p.get_future();

  // 스레드 수행
  thread t(worker, &p);

  // future가 준비될 때까지 기다림(블로킹)
  // =비동기 수행이 완료되었다는 플래그
  ft.wait();

  // wait() 리턴 = future 준비됨.(=비동기 수행 완료)
  // wait없이 get을 해도 wait과 같다.
  ft.get();

  t.join();
}
```

### shared_future
future의 get은 딱 한번만 호출할 수 있다. get()을 호출하면 future가 이동(move)되고 내부 멤버 future_status가 이동되기 때문이다.
멀티스레딩 환경에서는 여러 스레드에서 future에 get()을 할 필요가 있고 이런 경우에 shared_future를 사용하게 된다.
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

## 2. packaged_task

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

## 3. async
* 비동기 수행
  * std::launch::async : 새 스레드 생성, 바로 비동기 수행
  * std::launch::deferred : 현재 스레드에서, get() 호출 시 수행
* std::future 리턴

### async 예제
```c++
int AsyncTask(int a) {
  return a + 2;
}

int main() {
  int a = 1;

  // launch::async
  // 새 스레드 생성, 바로 비동기 수행
  future<int> ft1 = std::async(std::launch::async, AsyncTask, a);

  // launch::deferred
  // 현재 스레드에서, 아직 수행되지 않는다, 이후 get() 호출 시 수행.
  future<int> ft2 = std::async(std::launch::deferred, AsyncTask, a);

  // 메인 스레드 작업 중, AsyncTask 내용 수행 안됨.

  // get() 호출, 이 시점에 AsyncTask 내용 수행.
  ft2.get();

  // 추가적으로 async()를 사용할 때 async()의 리턴값을 받아주지 않는다면,
  // "future<T> ft ="에 해당하는 부분이 없이 async()만 호출한다면
  // 비동기로 수행되지 않고 코드가 적힌 순서대로 수행된다.
}
```

참고 : <br/>
https://modoocode.com/284#google_vignette <br/>

