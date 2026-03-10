# Move Semantics, 이동 의미론
객체 이동. 객체를 전달할 때 복사를 줄이고 소유권을 변경하는 방식을 장려하여 성능을 올리는 방식(기존 복사 방식 대비).

## 예를 통해 이해
```c++
class MyString {
public:
  // 복사 생성자
  // 원본의 데이터를 현재 객체에 복사
  MyString(const MyString& other) {
    size = other.size;
    data = new char[size];
    std::copy(other.data, other.data + size, data);
  }

  // 이동 생성자
  // 원본의 데이터의 소유권(포인터)을 현재 객체로 옮기고
  // 원본의 포인터는 제거
  MyString(MyString&& other) noexcept {
    size = other.size;
    data = other.data;

    other.size = 0;
    other.data = nullptr;
  }

  // 복사 대입 연산자
  MyString& operator=(const MyString& other) {
    if (this == &other)
      return *this;

    delete data;

    size = other.size;
    data = new char[size];
    std::copy(other.data, other.data + size, data);

    return *this;
  }

  // 이동 대입 연산자
  MyString& operator=(MyString&& other) noexcept {
    if (this == &other)
      return *this;

    delete data;

    size = other.size;
    data = other.data;

    other.size = 0;
    other.data = nullptr;

    return *this;
  }
public:
  char* data;
  int size;
};

int main() {
  MyString str1("abcd");

  // 복사 생성자, 이동 생성자
  MyString str2(str1);
  MyString str3(std::move(str1));

  // 복사 대입 연산자, 이동 대입 연산자
  MyString str4;
  str4 = str1;

  MyString str5;
  str5 = std::move(str1);
}
```

### 객체 이동?
복사 생성자, 복사 대입 연산자 말고 이동 생성자, 이동 대입 연산자를 통해 전달하는 거.

### std::move?
lvalue(좌측값)를 rvalue(우측값)로 바꿔주는 것.<br/>
이동 생성자, 이동 대입 연산자 함수의 매개변수 타입 캐스팅.<br/>

### 사용 이유?
원본이 유지될 필요가 없을 때 동적할당된 부분은 최대한 복사를 줄여 비용을 아끼기 위해.

### 그 밖에..
이동 연산자에 noexcept 붙이는 이유: vector가 move 대신 copy 사용.

<br/><br/>
<hr/>

# 레퍼런스

```c++
string func1() {
  string str = "func1";
  return str;
}
string& func2() {
  string str = "func2";
  return str;
}
string&& func3() {
  string str = "func3";
  return move(str);
}

int main() {
  string str1 = "main1";
  string str2 = "main2";
  string str3 = "main3";

  str1 = func1();
  cout << str1 << endl;

  str2 = func2();
  cout << str2 << endl;

  str3 = func3();
  cout << str3 << endl;
}

결과:
func1;
```
T - 복사본, 임시값, 우측값<br/>
T& - 원본, 좌측값<br/>
T&& - 원본, 우측값<br/>

T로 리턴하면 복사 발생, 성능적 측면을 고려할 때 T&, T&&를 사용.<br/>
댕글링 포인터를 주의하며 사용해야 한다.<br/>
위의 예제에서 func2, func3은 댕글링.<br/>

stl 컨테이너는 조회(const T&), 수정(T&)로 사용한다.
