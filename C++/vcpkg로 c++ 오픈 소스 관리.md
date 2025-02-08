# vcpkg?
Microsoft에서 제공하는 C/C++ 오픈 소스 패키지 툴 (오픈 소스 라이브러리를 프로젝트에 쉽게 추가하고 관리하게 해줌) <br/>

#### 간략 설명
두 가지 방법(클래식 모드, 매니페스트 모드)으로 나누어서 사용한다. (권장되는 사항은 매니페스트)
클래식(classic) 모드 - 다운 받은 오픈 소스를 여러 프로젝트에서 사용할 수 있도록 통합 관리
매니페스트(manifest) 모드 - 다운 받은 오픈 소스를 특정 프로젝트에서만 사용하게 관리
사용하는 빌드 시스템에 따라 사용법이 다른데 여기서는 MSBuild(비주얼스튜디오)를 사용하는 것을 전제로 한다.
vcpkg 설치와 사용할 오픈 소스 검색은 공통사항이며 후에 사용하고자 하는 방법에 따라 클래식 모드, 매니페스트 모드 설명란 참고

#### 지원 환경
운영 체제 : Windows, Linux, macOs <br/>
빌드 시스템 : MSBuild, CMake <br/>

# vcpkg 설치
1) 다운로드<br/>
https://github.com/microsoft/vcpkg 에서 깃이나 직접 다운로드 후 bootstrap-vcpkg.bat 파일을 실행하여 빌드<br/>

2) 환경 변수 설정<br/>
vcpkg.exe 파일이 생성되면 Path 환경 변수를 추가한다. (내 pc 우클릭 -> 속성 -> 고급 시스템 설정 -> 환경 변수 -> 시스템 변수 탭 -> Path 변수 편집 -> 새로 만들기 -> vcpkg.exe 실행 파일이 있는 폴더 입력)<br/>

3) 비주얼스튜디오 사용하는 경우<br/>
CMD에서 vcpkg가 설치된 경로에서 vcpkg integrate install을 입력해서 vcpkg를 비주얼 스튜디오에 통합 시킨다.

# 사용할 오픈 소스 검색 
1) 설치된 vcpkg로 검색 
vcpkg search 라이브러리명

2) 웹에서 검색
https://vcpkg.io/en/packages?query= 접속해서 원하는 라이브러리 검색

# 클래식 모드



# 매니페스트 모드

#### 초기 설정
1) 프로젝트에서 매니페스트 모드 활성화
비주얼스튜디오 -> 프로젝트 우클릭 -> 속성 -> vcpkg -> use manifest mode -> yes로 변경
2) 매니페스트 파일 생성 (vcpkg.json)
비주얼스튜디오 -> 도구 -> 명령줄 -> 개발자 명령 프롬프트 -> vcpkg new --application 명령 입력 (vcpkg.json, vcpkg-configuration.json, 이 2개 파일이 솔루션 폴더에 생성)

#### 라이브러리 설치(이 예제에서 사용할 오프 소스 = fmt)
1) 매니페스트 파일에 종속성(Dependency) 추가
CMD상에서 매니페스트 파일이 존재하는 경로에서 아래의 명령 형식을 입력하여 원하는 종속성을 추가한다.
vcpkg add port fmt

2) 설치
매니페스트 모드가 활성화된 프로젝트에서 빌드를 하면 MSBuild가 매니페스트 파일을 참조하여 명시된 종속성을 자동으로 설치한다.
다만, 실사용을 해보면 빌드가 굉장히 느려지기 때문에 필요한 오픈 소스를 설치하고 난 후라면 평소에는 종속성을 설치하는 기능을 Off하고 또 다른 오픈 소스를 추가할 필요가 있을 때 다시 On을 하는 것이 옳아 보인다.
프로젝트 속성 -> vcpkg -> install vcpkg dependencies

3) 설치되는 경로
솔루션폴더\vcpkg_installed\





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







<hr/>
출처 : <br/>
https://eachan.tistory.com/182 - CMake <br/>
https://luncliff.github.io/vcpkg-registry/vcpkg-for-kor.html - 한국어로 된 깊은 설명 <br/>
https://ralpioxxcs.github.io/post/vcpkg/vcpkg_1/ - <br/>

https://github.com/microsoft/vcpkg-docs/blob/main/vcpkg/consume/manifest-mode.md - MS 매니페스트 모드<br/>
- MS 클래식 모드<br/>
