### 쉽게
VM 같은 거.

### 윈도우에서 도커 사용
Docker는 리눅스 환경에서 돌아간다. 이를 윈도우 환경에서 돌리려면 WSL이 필요하다.<br/>
WSL 설치 후 도커 데스크톱 설치.<br/>
PreUser(개인용), AllUsers(윈도우 전체 사용자용) 중 개인용으로 설치.<br/>

* WSL (Windows Subsystem for Linux)
MS의 윈도우에서 리눅스를 사용하는 기술.<br/>
다른 VM과의 차이점은 호스트 os 공간과 별개로 잡는 것이 아니라 공유.<br/>

### 도커 감잡기 예제
```
// 도커 설치 후 테스트
docker run hello-world
// 도커가 pc에 hello-world라는 이미지를 찾고 없으면 인터넷(docker hub)에서 다운로드(pull)한 뒤 실행.
// 화면에 Hello from Docker! 메시지가 출력되면 성공


// 웹서버 띄우기
docker run -d -p 8080:80 --name my-web nginx
// -d 백그라운드 실행
// -p 8080:80 내 pc 8080 포트를 컨테이너 80포트로 연결
// --name my-web 이 컨테이너의 이름을 my-web으로 설정

docker stop my-web // 컨테이너 종료
docker rm my-web // 컨테이너 삭제
```

### 그래서 도커는?
한 pc의 상황인 척하는 프로세스.

