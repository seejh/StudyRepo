API(Application Programming Interface)

# Native API
간단히 말해서 운영체제(OS)나 플랫폼이 직접 제공하는 기본 프로그래밍 인터페이스를 뜻한다.<br/>

### 말로 설명하기 보다 예시로 설명
Win32 API vs Native API

### Win32 API
* CreateFile, WriteFile 등의 함수는 내부적으로 NtCreateFile 같은 Native API를 사용하는 래퍼.
* 사용자 친화적인 상위 레벨 API
* 사용하기 쉽고 문서화가 잘 되어 있음

```c++
#include<iostream>
#include<windows.h>
#pragma comment(lib, "ws2_32.lib")

int main() {
  // 파일 생성
  HANDLE file = CreateFileA(
    "example.txt", // 파일 이름
    GENERIC_WRITE, // 쓰기 권한
    NULL, // 보안 속성
    CREATE_ALWAYS, // 새 파일 생성
    FILE_ATTRIBUTE_NORMAL, NULL
  );

  if (file == INVALID_HANDLE_VALUE) {
    cerr << "파일 생성 실패" << endl;
    return 0;
  }

  // 내용 기입
  const char* text = "Hello from Win32 API";
  DWORD written;
  WriteFile(file, text, strlen(text), &written, NULL);

  // 마무리, 정리
  CloseHandle(file);
  cout << "file Create OK" << endl;
}
```

### Native API
* 훨씬 복잡하고 매개변수 많음
* Win32 API보다 직접적이고 빠르지만, 문서화되지 않았고 Windows 버전에 따라 변할 수 있음

```c++

#include<winternl.h>
#pragma comment(lib, "ntdll.lib")

// 아래 설명
extern "C" NTSTATUS NTAPI NtWriteFile(
  HANDLE FileHandle,
  HANDLE Event,
  PIO_APC_ROUTINE ApcRoutine,
  PVOID ApcContext,
  PIO_STATUS_BLOCK IoStatusBlock,
  PVOID Buffer,
  ULONG Length,
  PLARGE_INTEGER ByteOffset,
  PULONG Key
);

int main() {
  HANDLE file;
  IO_STATUS_BLOCK ioStatusBlock;

  UNICODE_STRING fileName;
  RtlInitUnicodeString(&fileName, L"\\??\\C:\\example_native.txt");

  OBJECT_ATTRIBUTES objAttr;
  InitializeObjectAttributes(&objAttr, &fileName, OBJ_CASE_INSENSITIVE, NULL, NULL);

  NTSTATUS status = NtCreateFile(
    &file,
    FILE_GENERIC_WRITE,
    &objAttr,
    &ioStatusBlock,
    NULL,
    FILE_ATTRIBUTE_NORMAL,
    0,
    FILE_OVERWRITE_IF,
    FILE_SYNCHRONOUS_IO_NONALERT,
    NULL,
    0
  );

  if (status != 0) {
    cerr << "error:0x" << std::hex << status << endl;
    return 0;
  }

  const char* msg = "Hello from Native API";
  ULONG written;
  status = NtWriteFile(file, NULL, NULL, NULL, &ioStatusBlock, (PVOID)msg, (ULONG)strlen(msg), NULL, NULL);

  NtClose(file);
  cout << "Native API로 파일 생성 완료" << endl;
}
```

### Win32 API 내부 호출 관계
CreateFile() (Win32) -><br/>
CreateFileInternal() (Kernel32 내부) -><br/>
NtCreateFile() (Native API, ntdll.dll) -><br/>
ZwCreateFile() (커널 모드로 트랩) -><br/>
NtCreateFile() (ntoskrnl.sys 내부 커널 함수)<br/>

Win32 API는 단지 Native API를 감싸는 래퍼.<br/>

### 코드 설명

#### 1. extern "C" NTSTATUS NTAPI NtWriteFile()
이 부분이 없으면 "오류(활성) E0020 식별자 "NtWriteFile"이(가) 정의되어 있지 않습니다."와 같이 에러가 난다.<br/>

Native API를 직접 사용할 때 생기는 전형적인 상황으로 NtWriteFile 같은 Native API 함수 선언이 포함되어 있지 않기 때문에 오류가 발생한다.
이 함수들은 Windows 내부용 함수로, 보통 ntdll.dll 안에 들어 있지만 공식 헤더에서는 비공개이다. <br/>

Visual Studio에서 선언이 빠져 있기 때문에 직접 선언을 추가해주는 것.

#### 2. 0xc0000022 에러
NTSTATUS 0xc0000022 = STATUS_ACCESS_DENIED = 접근 권한이 없어서 요청 거부.<br/>
관리자 권한 실행.(디버깅 중이면 vs를 관리자 권한 실행 후 디버깅)


