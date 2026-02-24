# future, promise, async, packaged_task
* 간단한 비동기 수행
* c++ 11 표준 라이브러리

## future, promise
```c++


```

## packaged_task

## async
* 간단한 스레드 생성
* std::future 리턴
```c++
int AsyncTask(int a) {
  return a + 2;
}

int main() {
  int a = 1;

  // launch::async
  // 새로운 스레드를 생성해서 바로 비동기 수행
  future<int> ft1 = std::async(std::launch::async, AsyncTask, a);

  // launch::deferred
  // 현재 스레드에서 이후 get이 호출되는 시점에 수행
  future<int> ft2 = std::async(std::launch::deferred, AsyncTask, a);

  // 메인 스레드 수행 중, AsyncTask 수행 안 되고 있음

  ft2.get(); // 이 시점에 수행

  // std::async를 수행할 때 리턴을 받아주지 않는다면
  // => async 앞에 "future<> ft ="를 하지 않는다면
  // 그냥 코드가 적힌 순서대로 수행된다.
}
```

