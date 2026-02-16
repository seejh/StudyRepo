
# Move Semantics (이동 의미론)
자원을 복사하는 것이 아니라 이동시킴으로써 복사 비용을 아껴 성능을 향상시키는 것.

## 예를 통해 이해
```c++
class MyString {
public:
  // 
  MyString(const char* str) {
    size = strlen(str);
    data = new char[size + 1];
    std::copy(str, str + size, data);
  }

  // 복사 생성자
  // 본인 공간을 따로 만들고 원본의 내용을 복사
  MyString(cosnt MyString& other) {
    size = other.size;
    data = new char[size];
    std::copy(other.data, other.data + size, data);
  }
  // 이동 생성자
  // 원본에서 포인터를 제거하고 본인이 포인터를 가짐 (=소유권 이전)
  MyString(MyString&& other) noexcept {
    size = other.size;
    data = other.data;

    other.data = nullptr;
    other.size = 0;
  }

  // 복사 대입 연산자
  // 본인 공간을 따로 만들고 원본의 내용을 복사
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
  // 원본에서 포인터를 제거하고 본인이 포인터를 가짐 (=소유권 이전)
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
  // 기본 생성자
  MyString str1("abcdef");

  MyString str2(str1);          // 복사 생성자
  MyString str3(std::move(str1)); // 이동 생성자
}
```



