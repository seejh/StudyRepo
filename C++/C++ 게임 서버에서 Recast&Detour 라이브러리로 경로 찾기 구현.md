
# Recast & Detour ?
Recast & Detour는 AI 경로 찾기 알고리즘을 구현하기 위한 오픈 소스 라이브러리.<br/>
Recast는 네비게이션 메시지를 생성, Detour는 이 메시지를 사용해 실시간 경로 계산.<br/>
이 두 라이브러리는 AI 캐릭터가 장애물을 피하며 목표 지점으로 이동하는데 필요한 핵심 기능을 제공한다.<br/>

### Recast
Recast는 3D 환경에서 네비게이션 메시지를 생성하는 라이브러리로, 맵의 장애물을 고려하여 AI가 이동할 수 있는 영역을 정의한다.<br/>
이 메시지는 AI 캐릭터가 이동할 수 있는 경로를 계산하는데 중요한 역할을 한다.

### Detour
Detour는 Recast가 생성한 네비게이션 메시지를 기반으로 경로를 계산하는 라이브러리.<br/>
실시간으로 경로를 찾으며, 장애물 회피 기능도 지원한다.<br/> 
Detour는 목표 지점까지의 최적 경로를 계산, 경로가 변경되거나 장애물이 나타났을 때 즉시 경로를 재계산한다.<br/>

# 프로젝트에 통합
TODO : 추가 방법과 샘플 확인법 추가 해야한다.
소스에 직접 통합, vcpkg 종속성 관리자를 통한 설치

# 프로젝트에 통합 후 시작
~~~c++
#include"Recast.h"
#include"DetourNavMesh.h"
#include"DetourNavMeshQuery.h"
~~~
Recast & Detour가 프로젝트에 성공적으로 통합됨. 이후부터는 네비게이션 메시지 생성, 경로 찾기 기능 구현

# 네비게이션 메시지 생성
Recast를 사용하여 AI가 이동할 수 있는 영역을 정의하는 네비게이션 메시지를 생성하는 과정은 경로 찾기의 첫 단계이다.<br/>
이 메시지는 3D 환경에서 AI 캐릭터가 이동할 수 있는 공간을 나타내며, 장애물과 이동 불가능한 영역을 고려하여 생성된다.

#### 환경 설정
Recast를 사용하기 전에 먼저 3D 환경을 설정해야 한다. 주로 .obj, .fbx 등의 형식으로 제공되는 3D 모델을 사용하여 환경을 정의한다.<br/>
이 모델에는 지형 정보, 장애물 등이 포함되어 있어야 하며, Recast는 이를 바탕으로 네비게이션 메시지를 생성한다.<br/>

#### 네비게이션 메시지 생성 함수
Recast의 핵심 함수는 rcBuildNavMesh이다. 이 함수는 3D 환경에서 AI가 이동할 수 있는 네비게이션 메시지를 생성한다. 생성된 메시지는 장애물을 피하고, 경로를 계산하는데 사용된다. 아래는 네비게이션 메시지 생성을 위한 예시 코드이다.
~~~c++
rcConfig cfg;
cfg.cs = 0.3f;  // 셀 크기
cfg.ch = 0.2f;  // 셀 높이
cfg.width = 512; // 그리드 너비
cfg.height = 512; // 그리드 높이

// 3D 환경을 로드하고 메시 생성
rcHeightfield* heightfield = rcAllocHeightfield();
if (!rcCreateHeightfield(context, *heightfield, cfg.width, cfg.height, &verts[0], numVerts, cfg.cs, cfg.ch))
{
    printf("Heightfield creation failed.\n");
    return;
}

// 네비게이션 메시지 생성
rcPolyMesh* polyMesh = rcAllocPolyMesh();
rcBuildPolyMesh(context, *heightfield, cfg, *polyMesh);

rcNavMesh* navMesh = rcAllocNavMesh();
if (!rcBuildNavMesh(context, *polyMesh, cfg, *navMesh))
{
    printf("NavMesh creation failed.\n");
    return;
}
~~~

#### 네비게이션 메시지 확인
생성된 네비게이션 메시지는 디버깅 도구를 사용하여 확인할 수 있다. Recast는 메시를 시각적으로 확인할 수 있는 기능을 제공하여, 생성된 경로와 장애물 회피가 제대로 이루어지는지 점검할 수 있다. 이 과정은 AI가 실제 환경에서 경로를 잘 계산하는지 확인하는 중요한 단계이다.

# Detour를 활용한 경로 찾기
Detour는 Recast가 생성한 네비게이션 메시지를 바탕으로 AI 캐릭터의 경로를 계산하는 라이브러리이다. Detour는 실시간 경로 찾기와 장애물 회피 기능을 지원하며, 주어진 시작 지점과 목표 지점 간의 최적 경로를 찾아낸다.


<hr/>
출처 : <br/>
https://ko.ittrip.xyz/c-plus-plus/cpp-game-server-recast-detour-pathfinding
https://github.com/recastnavigation/recastnavigation

<br/><br/><br/><br/><br/><br/>

<hr/>

# Recast Navigation 샘플 데모 사용(일단 여기다 정리, 추후 수정 필요)
https://github.com/recastnavigation/recastnavigation/tree/main 에서 RecastNavigation 다운로드<br/>
다운 후 솔루션 파일 생성을 해야 하는데 선행 되어야 할 것들이 있다.

#### SDL 라이브러리
https://github.com/libsdl-org/SDL 에서 SDL(Simple Directmedia Layer) 다운로드<br/>
다운 후 RecastDemo/Contrib 에 압축을 풀어준다.

#### premake로 솔루션 파일 생성
https://premake.github.io/ 에서 premake를 다운로드<br/>
RecastDemo 폴더에 위치시킨 후 CMD 상에서 아래와 같은 형식으로 명령을 입력한다.<br/>
![image](https://github.com/user-attachments/assets/4885bb22-8bb0-43cd-b53d-9103c4921d6b)

해당 명령 실행 후 RecastNavigation의 솔루션 파일이 생긴다.
해당 파일을 비주얼 스튜디오로 실행한 후 RecastDemo를 시작 프로젝트로 설정 후 빌드 후 실행한다.


출처 : <br/>
https://yunus-lab.tistory.com/13  - 샘플 사용 내용<br/>

