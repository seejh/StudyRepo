
# vcpkg란?
Microsoft에서 제공하는 오픈소스 C/C++ 패키지 툴<br/>
C++ 오픈 소스 라이브러리 설치, 링크를 간단하게 해준다.<br/>
내부적으로 라이브러리의 소스 코드를 다운 받아서 직접 컴파일하는 방식을 사용한다.<br/>

#### 기본 설명
다운 받은 오픈 소스 라이브러리를 사용할 특정 프로젝트에 추가(Manifest Mode)하거나 통합(Classic Mode)하여 추가시킬 수 있다.
여기서는 IDE에 통합하여 사용하는 방법으로 설명, 특정 프로젝트에만 추가하는 방법은 따로 학습.

#### 지원
운영체제 : windows, linux, maxos<br/>
빌드 시스템 : MSBuild, CMake<br/>

# vcpkg 설치
https://github.com/microsoft/vcpkg 에서 깃이나 직접 다운로드 후 bootstrap-vcpkg.bat 파일을 실행하여 빌드<br/>
vcpkg.exe 파일이 생성되면 Path 환경 변수를 추가한다. (내 pc 우클릭 -> 속성 -> 고급 시스템 설정 -> 환경 변수 -> 시스템 변수 탭 -> Path 변수 편집 -> 새로 만들기 -> vcpkg.exe 실행 파일이 있는 폴더 입력)<br/>

# vcpkg와 visual studio 연동
CMD 상에서 vcpkg integrate install 입력. visual studio 에서 vcpkg에 설치된 라이브러리를 자동으로 인식한다.

# 패키지 검색, 설치
#### 검색
CMD 상에서 검색할 수 있고, 웹에서 검색할 수 있다.

1)CMD 상에서 검색<br/>
CMD 상에서 vcpkg search 라이브러리
![image](https://github.com/user-attachments/assets/757056b4-9173-4dd5-8206-8c7e1a81b722)

2)웹에서 검색<br/>
웹에서 https://vcpkg.io/en/packages?query= 접속 후 라이브러리 검색
![image](https://github.com/user-attachments/assets/edd16615-9763-47a1-b2e3-736036a5f537)

#### 설치
visual studio 영어 언어 팩이 없으면 패키지 설치가 되지 않는다고 한다. 없으면 visual studio installer에서 언어 팩 란에서 영어를 찾아서 설치.<br/>
CMD 상에서 아래와 같은 형식으로 입력하여 다운 받는다.
사용 환경(triplets)은 명시하지 않아도 되지만, 기본값은 x86-windows 이다.
~~~
vcpkg install 라이브러리명:사용 환경(triplets)
~~~

![image](https://github.com/user-attachments/assets/da6a2754-ec1b-4fae-a184-69d5a740b984)

# 설치된 패키지 사용
설치된 라이브러리는 아래 경로에 설치되며 비주얼 스튜디오 상에서 #include 라이브러리명으로 찾아서 사용.<br/>
경로 : vcpkg 디렉토리\installed\x64-windows\ <br/>


vcpkg integrate remove

<hr/>
출처 : <br/>
https://ralpioxxcs.github.io/post/vcpkg/vcpkg_1/ <br/>
https://jacking75.github.io/Cpp-0501/ - <br/>
