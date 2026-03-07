# future, promise, async, packaged_task
* 비동기 수행
* c++ 11 표준 라이브러리

## async
* std::thread와 비슷하다.
* std::future 리턴.
* 비동기 수행 (옵션에 따라 다른 수행)
  * launch::async
  * launch::deferred

### 예제
```c++
#include<future>

int AsyncTask(int n) { return n + 1; }

int main() {
 // std::launch::async 옵션
 // 즉시 새 스레드 생성하여 비동기 수행
 future<int> ft1 = std::async(std::launch::async, AsyncTask, 1);
 int result1 = ft1.get();

 // std::launch::deferred 옵션
 // 스레드 생성없이, 현 스레드에서, get() 호출 시점에 수행
 future<int> ft2 = std::async(std::launch::deferred, AsyncTask, 2);

 // 아직 수행 안됨, 메인 스레드에서 다른 작업 중

 ft2.get(); // 이 시점에 수행

 // async() 함수를 사용할 때 리턴값을 받아주지 않는다면
 // 비동기로 동작하지 않는다.
}
```

## future, promise
* 비동기 수행하는 스레드 간 값 전달
* promise: 값 전달
* future: 값 받음

### 예제
```c++
void AsyncTask(promise<int>* pm) {
 this_thread::sleep_for(chrono::seconds(5));

 // promise 값 설정
 pm->set_value(2222);
}

int main() {
 promise<int> pm;
 future<int> ft = pm.get_future();

 // 타 스레드에서 비동기 수행
 thread t(AsyncTask, &pm);

 // future 값 받기
 ft.get();

 t.join();
}
```

## packaged_task
= *콜백 함수


```c++
ㅇ
```

* 콜백 함수:
다른 함수에 인자(parameter)로 전달되어,
해당 함수의 작업이 완료된 후나 특정 이벤트 발생 시,
나중에 호출(call back)되는 함수


