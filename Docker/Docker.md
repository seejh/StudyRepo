VM 같은 거.

Docker는 리눅스 환경에서 돌아간다. 이를 윈도우 환경에서 돌리려면 WSL이 필요하다.<br/>
WSL 설치 후 도커 데스크톱 설치.<br/>
PreUser(개인용), AllUsers(윈도우 전체 사용자용) 중 개인용으로 설치.<br/>

도커란?<br/>
감잡기<br/>

// 도커 테스트
docker run hello-world
// 도커가 pc에 hello-world라는 이미지를 찾고 없으면 인터넷(Docker Hub)에서 다운로드(pull)한 뒤 실행.
// 화면에 Hello from Docker! 메시지가 출력되면 성공

// 웹서버 띄우기
docker run -d -p 8080:80 --name my-web nginx
// -d 백그라운드 실행
// -p 8080:80 내 pc 8080포트를 컨테이너 80 포트로 연결
// --name my-web 이 컨테이너의 이름을 my-web 으로 설정

docker stop my-web // 컨테이너 종료
docker rm my-web // 삭제

한 pc의 상황인 척하는 프로세스.

