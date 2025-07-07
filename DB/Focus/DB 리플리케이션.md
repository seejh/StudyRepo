Mysql 8.0 버전 이상, 스토리지 엔진은 InnoDB 기준

# DB Replication?
DB 복제하는 거. 하나 잘 쓰면 되지 않나? 왜 같은 거 여러개 씀? <br>
상황 예시로 이해 <br/>

## Case1
사용자가 요청을 보냈는데 데이터베이스가 응답하지 않는다. <br/>
설상가상 다시 재가동도 되지 않는다. <br/>
![image](https://github.com/user-attachments/assets/cf5a6bd8-afe2-4a61-90ba-d8ddf6144446) <br/>

## Case2
서비스 사용자가 늘어서 요청 트래픽이 증가<br/>
트래픽의 부하 분산이 필요한 상황 <br/>
이런 경우 DB를 스케일업(DB 머신 성능, DB 소프트웨어 성능?)을 해도 되겠지만 한계는 있다. <br/>
![image](https://github.com/user-attachments/assets/427a625c-1776-4d08-82bd-1adefcd500f0) <br/>

이러한 경우들을 해결하기 위해 리플리케이션이 등장한다.


출처 : <br/>
https://www.youtube.com/watch?v=NPVJQz_YF2A <br/>
<hr/><br/><br/>

