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

struct Base { // 부모 클래스에 가상함수가 있으면 구분가능 (= 가상함수가 있어야 구분이 가능하다.)
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

예제3) 
```c++
#include<iostream>
#include<map>
#include<typeindex>

int main() {
  map<type_index, string> m =
  {
    { typeid(int), "int" }, { typeid(double), "double" }
  };

  cout << m[typeid(int)] << endl; // 출력 : int
}
```


출처 : <br/>
https://jacking75.github.io/C++11_typeInfo_typeIndex/
