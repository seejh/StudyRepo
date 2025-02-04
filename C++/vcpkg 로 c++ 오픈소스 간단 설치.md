
# vcpkg란?
Microsoft에서 제공하는 오픈소스 C/C++ 패키지 매니저이다. C++ 라이브러리 설치, 링크를 간단하게 해준다.
파이썬의 pip, Node.js의 npm, Rust의 cargo 등과 비슷한 기능을 한다.

운영체제는 Windows, Linux, maxOS를 지원.
빌드 시스템은 MSBuild 와 CMake 를 지원.

내부적으로 라이브러리의 소스 코드를 다운 받아서 직접 컴파일하는 방식을 사용한다.

# vcpkg 설치
// 여기서는 깃을 이용해서 설치를 진행한다.(미리 깃이 설치되어 있어야 한다.)
1.cmd 상에서 vcpkg를 설치할 위치로 이동 후 vcpkg 리포지토리를 복제한다.
~~~
git clone https://github.com/microsoft/vcpkg
~~~

2.vcpkg 설치
~~~
.\vcpkg\bootstrap-vcpkg.bat
~~~
vcpkg.exe 파일이 생성되면 Path 환경 변수(경로\vcpkg)를 추가한다.

# vcpkg와 visual studio 연동
둘 다 Microsoft 것이라서 연동이 쉽다. CMD 상에서 아래 명령줄을 쳐주면 된다.
~~~
vcpkg integrate install
~~~
MSVC(Microsoft Visual C++) 가 vcpkg 에 설치된 라이브러리를 자동으로 인식한다.

# 패키지 검색, 설치

1.검색
vcpkg search 검색어
위와 같은 형식으로 입력하면 vcpkg 공개 저장소에 등록된 패키지를 검색해준다.
검색 결과에서 왼쪽 열에 있는 패키지명을 기억하면 된다. 또는 웹에서 검색 가능.
웹 경로(https://vcpkg.io/en/packages.html?query=)
~~~
PS> vcpkg search xml
...
rapidxml                 1.13-4           RapidXml is an attempt to create the fastest XML parser possible, while re...
rapidxml-ns              1.13.2           RapidXML with added XML namespaces support.
tidy-html5               5.7.28-2         Tidy tidies HTML and XML. It can tidy your documents by itself, and develo...
tinyxml                  2.6.2-7          A simple, small, minimal, C++ XML parser that can be easily integrating in...
tinyxml2                 8.0.0-1          A simple, small, efficient, C++ XML parser
...
The search result may be outdated. Run `git pull` to get the latest results.

If your library is not listed, please open an issue at and/or consider making a pull request:
    https://github.com/Microsoft/vcpkg/issues
~~~

2.설치
visual studio 영어 언어 팩이 없으면 패키지 설치가 되지 않는다.
없다면 visual studio installer를 켜서 언어 팩 에서 영어를 찾아서 설치해야한다.

패키지는 두 가지 모드로 설치 가능하다.
Classic Mode : 모든 프로젝트에 사용 가능하도록 전역 설치
Manifest Mode : 특정 프로젝트에 의존성을 명시하여, 그 프로젝트 폴더 내 설치

Classic Mode (전역 라이브러리 설치)
vcpkg install 패키지명:실행 환경, 이와 같은 형식으로 입력하면 된다. (ex, vcpkg install rapidxml x64-windows)

Manifest Mode (프로젝트별 라이브러리)
vcpkg 공식 Docs에서는 Classic 보다는 이 방법을 사용할 것을 권장한다.
프로젝트 디렉토리(MSBuild) 또는 솔루션 디렉토리(MSBuild, CMake) 에 아래와 같은 형식의 vcpkg.json 파일 생성
~~~json
{
  "$schema": "https://raw.githubusercontent.com/microsoft/vcpkg/master/scripts/vcpkg.schema.json",
  "name": "my-app",
  "version": "0.0.1",
  "dependencies": [
    "fmt",
    "pugixml",
    "nlohmann-json"
  ]
}
~~~
name - 
version - 
dependencies - 설치할 라이브러리

MSBuild 기반 프로젝트의 경우, 프로젝트 속성 -> 구성 속성 -> vcpkg -> Use Vcpkg Manifest를 Yes로 바꿔줘야 한다.
구성 속성에서 vcpkg 탭이 보이지 않을 경우, CMD 상에서 vcpkg integrate install 입력을 통해 visual studio와 연동부터 해야 한다.

# 설치된 패키지 사용

1.MSBuild에서 패키지 사용
MSBuild 기반 프로젝트에서는, 대부분의 경우 뭘 더 하지 않아도 바로 라이브러리의 헤더를 #include 해서 사용 가능.
#include 하는 순간, MSBuild 가 알아서 해당 패키지의 정적 라이브러리를 링크 해주며, 해당 라이브러리에 사용되는 DLL 파일이 자동으로 빌드 폴더로 
복사된다.


2.CMake에서 패키지 사용
CMake 기반 프로젝트의 경우, vcpkg.json 파일이 최상단 CMakeLists.txt 가 있는 디렉토리에 있어야 한다.
CMakeLists.txt 에 find_package(), target_link_libraries() 를 작성. 아래는 예시.
~~~
# 예시 (SDL2 오픈 소스 라이브러리)
find_package(SDL2 REQUIRED)
target_link_libraries(main PRIVATE SDL2::SDL2 SDL2::SDL2main)
~~~

cmake 를 호출할 때 툴체인 파일을 명시해준다.
~~~
cmake {project_dir} -DCMAKE_TOOLCHAIN_FILE={util_dir}\vcpkg\scripts\buildsystems\vcpkg.cmake
~~~
visual studio 를 사용한다면 빌드할 때 툴체인 파일 명시는 IDE 가 알아서 해준다.
이제 라이브러리의 헤더를 #include 해서 사용하면 되며, MSBuild 와 마찬가지로 DLL 파일이 자동으로 복사된다.

<hr/>
출처 : https://velog.io/@copyrat90/getting-started-with-vcpkg

