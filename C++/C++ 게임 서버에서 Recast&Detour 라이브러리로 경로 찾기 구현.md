
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

<hr/>
출처 : <br/>
https://ko.ittrip.xyz/c-plus-plus/cpp-game-server-recast-detour-pathfinding
https://github.com/recastnavigation/recastnavigation

