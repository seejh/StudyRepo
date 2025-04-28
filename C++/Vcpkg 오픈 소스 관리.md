// CMAKE 추가 필요

# vcpkg?
Microsoft에서 제공하는 C/C++ 오픈 소스 패키지 툴 <br/> 
오픈 소스 라이브러리를 프로젝트에 쉽게 추가하고 관리하게 해준다 <br/><br/>

두 가지 방법 (클래식 모드, 매니페스트 모드)으로 나누어서 사용한다. 권장되는 사항은 매니페스트라 한다. <br/>
사용하는 빌드 시스템에 따라 사용법이 다른데 여기서는 윈도우, MSBuild(비주얼스튜디오)를 사용하는 것을 전제로 한다. <br/>
visual studio 영어 언어 팩이 없으면 패키지 설치가 안 된다고 한다. <br/>

### 클래식(classic) 모드
다운 받은 라이브러리를 여러 프로젝트에서 사용할 수 있도록 통합 관리 <br/>
### 매니페스트(manifest) 모드
특정 프로젝트에서만 사용하겠다는 방식으로 다운 받은 라이브러리들을 해당 프로젝트 폴더 내에서 관리 <br/>

## vcpkg 설치
### 1. vcpkg 다운로드
https://github.com/microsoft/vcpkg 에서 다운로드 <br/>
설치할 경로에 압축을 풀고, bootstrap-vcpkg.bat 파일을 실행하여 빌드<br/>
(또는 visual studio installer를 통해서 설치도 가능하다)

### 2. 환경 변수 설정
1번의 결과로 vcpkg.exe 파일이 생성되면 Path 환경 변수를 추가 <br/>
내 pc 우클릭 -> 속성 -> 고급 시스템 설정 -> 환경 변수 -> 시스템 변수 탭 -> Path 변수 편집 -> 새로 만들기 -> vcpkg.exe 실행 파일 경로를 입력 <br/>
CMD에서 vcpkg 명령을 입력해서 뭔가 뜨면 환경 변수 설정이 된 것 <br/>

### 3. 비주얼 스튜디오에 통합
CMD에서 vcpkg가 설치된 경로에서 vcpkg integrate install을 입력해서 vcpkg를 비주얼 스튜디오에 통합 시킨다. <br/>
1번 절차에서 visual studio installer를 통해 설치했다면 필요없음 <br/>

## 사용 (예제로 사용할 라이브러리 fmt)
클래식 모드와 매니페스트 모드에 따라 사용 방법이 다르다 <br/>
### 1. 클래식 모드 사용 시
#### 사용할 라이브러리 검색
CMD 상에서 vcpkg search fmt를 하거나 <br/>
웹에서(https://vcpkg.io/en/packages?query=) 검색할 수 있다. <br/>
#### 라이브러리 다운로드
CMD상에서 아래 명령을 입력하면 \vcpkg 설치 경로\installed\ 경로에 설치된다. <br/>
vcpkg install fmt <br/>
#### 프로젝트에서 사용
#include<fmt/...> 하여 사용하면 된다.

### 2. 매니페스트 모드 사용
#### 프로젝트 매니페스트 모드로 변경
비주얼스튜디오 -> 매니페스트 모드를 사용할 프로젝트에서 프로젝트 우클릭 -> 속성 -> Use Vcpkg Manifest Yes로 변경 <br/>
비주얼스튜디오 -> 도구 -> 명령줄 -> 개발자 명령 프롬프트 -> vcpkg new --application 명령 입력 <br/>
위 명령을 입력하면 솔루션 폴더에 vcpkg.json, vcpkg-configuration.json, 2개 파일이 생성
##### 매니페스트 파일에 종속성 명시
CMD 상에서 매니페스트 파일이 존재하는 경로에서 아래 명령 형식을 입력하여 종속성을 명시 <br/>
vcpkg add port fmt <br/>
##### 프로젝트 빌드
프로젝트 우클릭 -> 속성 -> Install Vcpkg Dependencies를 Yes로 변경 후 프로젝트를 빌드하면 명시한 종속성을 자동으로 설치한다. <br/>
매니페스트 모드가 활성화된 프로젝트에서 빌드를 하면 MSBuild가 매니페스트 파일을 참조하여 명시된 종속성을 자동으로 설치한다. <br/>
다른 방법도 있을텐데 현재 이렇게 사용하고 있으며, Install Vcpkg Dependencies 옵션은 설치할 때 아니면 꺼두는 것이 좋아보인다. <br/>
라이브러리는 프로젝트 솔루션 폴더\vcpkg_installed\에 설치된다. <br/>

출처 : <br/>
https://github.com/microsoft/vcpkg-docs/blob/main/vcpkg/consume/manifest-mode.md - MS 매니페스트 모드<br/>
https://github.com/microsoft/vcpkg-docs/blob/main/vcpkg/consume/classic-mode.md - MS 클래식 모드<br/>
https://learn.microsoft.com/ko-kr/vcpkg/get_started/get-started-msbuild?pivots=shell-cmd - 비주얼 스튜디오, MSBuild 사용 <br/>
<hr/>
