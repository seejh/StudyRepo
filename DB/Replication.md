Replication이란?
두 개 이상의 DBMS를 이용하여 Master/Slave의 수직적 구조를 활용하여 DB의 부하를 분산시키는 기술이다.

![image](https://github.com/user-attachments/assets/6e7c5434-56bf-45d7-a317-f8d54026d63c)

어떻게(부하분산을)?
Master DB에서 Insert, Update, Delete 작업을 수행하고 Slave DB에서 Select 작업을 수행하는 것.
왜 Select 작업을 따로 빼는가? 보통 Select 작업이 시간이 많이 걸리기 때문이다. 


출처 : 
https://velog.io/@zpswl45/DB-Replication-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC
