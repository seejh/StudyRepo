
# Recast & Detour란 ?
Recast & Detour는 AI 경로 찾기 알고리즘을 구현하기 위한 오픈 소스 라이브러리.<br/>
Recast는 네비게이션 메시(NavMesh)를 생성, Detour는 이 메시를 사용해 실시간 경로 계산.<br/>
이 두 라이브러리는 AI 캐릭터가 장애물을 피하며 목표 지점으로 이동하는데 필요한 핵심 기능을 제공한다.<br/>

### Recast
Recast는 3D 환경에서 네비게이션 메시를 생성하는 라이브러리로, 맵의 장애물을 고려하여 AI가 이동할 수 있는 영역을 정의한다.<br/>
이 메시지는 AI 캐릭터가 이동할 수 있는 경로를 계산하는데 중요한 역할을 한다.

### Detour
Detour는 Recast가 생성한 네비게이션 메시를 기반으로 경로를 계산하는 라이브러리.<br/>
실시간으로 경로를 찾으며, 장애물 회피 기능도 지원한다.<br/> 
Detour는 목표 지점까지의 최적 경로를 계산, 경로가 변경되거나 장애물이 나타났을 때 즉시 경로를 재계산한다.<br/>



# 출처
https://ko.ittrip.xyz/c-plus-plus/cpp-game-server-recast-detour-pathfinding - 설명 <br/>
https://github.com/recastnavigation/recastnavigation - RecastNavigation 라이브러리 <br/>
https://www.slideshare.net/MUUMUMUMU/recast-detourpptx#33 - 설명 <br/>
<br/><br/><br/><br/><br/><br/>

<hr/>

# Recast Navigation 샘플 데모 사용(일단 여기다 정리, 추후 수정 필요)

# 설치
https://github.com/recastnavigation/recastnavigation/tree/main 경로에서 RecastNavigation을 직접 다운 후 압축 해제<br/>

#### SDL(Simple Directmedia Layer) 라이브러리 설치
2점 대 버전을 설치해야 한다.(ex https://github.com/libsdl-org/SDL/releases/tag/release-2.30.7) <br/>
![image](https://github.com/user-attachments/assets/724c24e9-6de1-40ce-8950-e82ed4068388)<br/>
다운 후 RecastDemo/Contrib 에 압축 해제, 설치된 "SDL-버전" 형식으로 되어 있는 폴더명을 SDL로 변경(/RecastDemo/Contrib/SDL/)<br/>

#### premake로 솔루션 파일 생성
https://premake.github.io/ 에서 premake를 다운 후 RecastDemo 폴더에 압축 해제 (premake5.exe 실행 파일 하나)<br/>
Path 환경 변수에 등록, CMD 상에서 아래와 같은 형식으로 명령을 입력한다.<br/>
![image](https://github.com/user-attachments/assets/425a0a1d-767f-40e3-aa65-d78dae31da3e)<br/>
해당 명령 실행 후 RecastDemo의 솔루션 파일 생성 (RecastDemo/Build/vs2022/ 경로에 생성)<br/>
해당 파일을 비주얼 스튜디오로 실행 -> RecastDemo를 시작 프로젝트로 설정 -> 빌드 -> 실행<br/>

# 사용
Sample_SoloMesh.cpp, NavMeshTestTool.cpp 이 두 파일 위주로 보면서 학습하는 것이 좋다.<br/>
자세한 설명은 아래 링크 참고<br/>

출처 : <br/>
https://yunus-lab.tistory.com/13  - 샘플 사용 내용<br/>
<br/><br/><br/><br/><br/><br/>

<hr/>

# RecastNavigation C++ 게임 서버에서 사용(Unreal, C++ 게임 서버)
#### 수순
1) Unity, Unreal에서 맵 정보를 .obj 파일로 추출
2) .obj 파일을 Recast로 렌더링 (RecastDemo에서 불러와서 렌더링)
3) Build 버튼을 눌러 네비게이션 메쉬 생성 후 .bin 파일로 추출
4) 서버에서 .bin 파일을 읽어 네비게이션 메쉬를 로드한 후 detour 라이브러리로 길 찾기 수행 <br/>

위의 수순 보다 더 간단하게 하려면 언리얼에서 네비 메쉬 자체를 뽑아서 .bin 파일로 담아 서버로 들고와서 detour를 수행<br/>

# Unreal에서 네비 메쉬 추출
언리얼 엔진에 네비게이션 메쉬 추출 플러그인을 추가 <br/>
본인은 언리얼4.27 버전을 쓰기에 아래 경로에서 외부 라이브러리를 다운 받아 플러그인으로 추가한다. <br/>
https://github.com/hxhb/ue4-export-nav-data <br/>
![image](https://github.com/user-attachments/assets/89267a26-9712-4108-85d6-ce44e74fc8ac) <br/>
추가하면 위와 같이 되며 Export Nav를 누르면 원하는 경로에 아래와 같이 파일로 추출한다. <br/>
![image](https://github.com/user-attachments/assets/d761617d-9d73-4a68-9010-2ae4fbcee7b6) <br/>




# 출처
https://blog.naver.com/PostView.naver?blogId=jokorat&logNo=223422345504 - <br/>


<hr/>

