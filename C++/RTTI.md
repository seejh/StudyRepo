RTTI (RunTime Type Information/Identification)
런타임에 타입 유추하는거

예제1)
```c++
#include<iostream>
#include<typeinfo>

class A {};
int main() {
  const type_info* ptr;
  ptr = &typeid(int);
  cout << ptr->name() << endl; // 출력 : int

  ptr = &typeid(A);
  cout << ptr->name() << endl; // 출력 : class A
}
```

예제2) 상속 관계의 클래스 타입 구분
```c++
#include<iostream>
#include<typeinfo>

struct Base { // Base가 되는 클래스에서 가상 함수가 없으면 구분을 못함
  virtual void p() {}
};
struct Cat : Base {};
struct Dog : Base {};

string GetName(Base& ref) {
  //
  decltype(auto) ti = typeid(ref);
  // == type_index ti = typeid(ref);

  if (ti == typeid(Cat))
    return "Cat";
  else if (ti == typeid(Dog))
    return "Dog";

  return "None";
}

int main() {
  Base base;
  cout << GetName(base) << endl;

  Cat cat;
  cout << GetName(cat) << endl;

  Dog dog;
  cout << GetName(dog) << endl;
}
```

출처 : <br/>
https://jacking75.github.io/C++11_typeInfo_typeIndex/
