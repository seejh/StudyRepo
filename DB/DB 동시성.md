# DB 동시성

# 1. 동시성 효과(정합성 문제)
동일한 데이터에 대해 여러 트랜잭션이 동시에 접근했을 때 발생하는 부작용 유형 <br/>
잠금 레벨에 따라 아래의 문제가 생기고 안 생기고 한다. <br/>
여기서는 간략하게 설명한 것으로 자세한 이해는 2. 잠금과 3. 잠금 레벨까지 읽어야
제대로된 이해가 가능하다.
## 2-1. Dirty Read
커밋되지 않은 내용을 읽는 경우 <br/>
한 트랜잭션에서 수정 후 커밋하지 않은 내용을 타 트랜잭션에서 읽을 때 발생하는 상황으로 아래와 같다. <br/>
* 트랜잭션1(T1)에서 레코드 A의 값을 2로 수정 (커밋이나 롤백을 통해 트랜잭션을 끝내지는 않음)
* 트랜잭션2(T2)에서 레코드 A를 조회 (변경된 2가 조회)
## 2-2. Non-Repeatable Read(일관되지 않은 읽기)
한 트랜잭션에서 동일한 레코드에 대해 여러번 조회를 할 때, 데이터의 내용이 일관되지 않을 경우(타 트랜잭션에 의해 변경되서) <br/>
* T1에서 레코드 A를 읽음 (값 40 조회, 트랜잭션 여전히 진행중)
* T2에서 레코드 A를 100으로 변경
* T1에서 레코드 A를 읽음 (값 100)
## 2-3. Phantom Read
한 트랜잭션에서 동일한 레코드에 대해 여러번 조회를 할 때, 없던 데이터가 생기거나 있던 데이터가 사라지는 경우
* T1에서 SELECT로 조회(3개 행이 조회가 됨, 트랜잭션 여전히 진행중)
* T2에서 INSERT 또는 DELETE
* T1에서 SELECT로 조회(행이 3개가 아님)

# 2. 잠금
## 1-1. 공유 잠금 (Shared Lock, S락, 읽기 잠금)
데이터를 읽을(=Read, SELECT) 때 사용하는 락 <br/> 
어느 트랜잭션에서 공유 잠금을 걸었을 때 타 트랜잭션에서 공유 잠금은 획득할 수 있지만 배타 잠금은 획득할 수 없다. <br/>
## 1-2. 배타 잠금 (Exclusive Lock, X락, 쓰기 잠금)
데이터를 쓸(=Write, INSERT, UPDATE, DELETE) 때 사용하는 락 <br/>
어느 트랜잭션에서 배타 락을 걸면 해제할 때까지 타 트랜잭션에서 read, write 할 수 없다. <br/>

# 3. DB 엔진의 격리 수준(잠금 레벨)
MSSQL 기준으로 설명 <br/>
Read UnCommitted, Read Committed, Repeatable Read, Serializable, Snapshot 이 있다. <br/>
어느 잠금 레벨을 설정하든간에 기본적으로 Write하는 상황에 배타 잠금은 걸린다. <br/>
## 3-1. Read UnCommitted 격리 수준
Read(SELECT)할 때 공유 잠금을 걸지 않는 단계 <br/>
```sql
-- T1 (트랜잭션1)

-- 트랜잭션 격리 레벨 Read UnCommitted로 설정
-- 한 번 설정하면 세션이 유지되는 동안 유지됨으로 계속 입력할 필요 없다.
set transaction isolation level read uncommitted

begin tran
update ttt set name='sss' where id=1

-- T2 (트랜잭션2)
set transaction isolation level read uncommitted
begin tran
select * from ttt
```
* T1에서 X락을 걸고 id=1인 레코드 수정
* T2에서 id=1인 레코드 조회(name=sss)
위의 수순으로 동작하며 T2에서 T1에서 작업했지만 커밋하지 않은 내용에 대해서 조회할 수 있다.
<br/><br/>

```sql
-- T1 
set transaction isolation level read uncommitted
begin tran
update ttt set name='sss' where id=1;

-- T2 
begin tran
update ttt set name='nnn' where id=1
commit
```
* T1에서 X락을 걸고 id=1인 레코드를 수정
* T2에서 id=1인 레코드에 X락을 걸려고 하지만 T1에서 점유하고 있어 대기
* 이후 T1에서 commit이나 rollback을 해주면 T2의 대기가 풀리고 진행
위의 수순으로 동작하며 T2에서 T1의 작업이 완료될 때까지 대기한다.
<br/><br/>

```sql
-- T1
set transaction isolation level read uncommitted
begin tran
update ttt set name='sss' where id=1

-- T2
set transaction isolation level read uncommitted
begin tran
update ttt set name='sss' where id=2
```
* T1에서 X락을 걸고 id=1인 레코드의 name을 수정(name=sss)
* T2에서 X락을 걸고 id=2인 레코드의 name을 수정(name=sss)
위의 수순으로 동작하며 T1은 X락을 레코드(id=1)에 걸었기에 T2에서 레코드(id=2)에 접근 가능

### Read Uncommitted 정리
타트랜잭션에서 수정 후 커밋하지 않은 내용을 조회할 수 있기에 Dirty Read가 발생한다. (그 외에도 Non-Repeatable Read, Phantom Read 발생 가능)

## 3-2. Read Committed 격리 수준
SQL Server 및 대다수 RDBMS의 기본 잠금 레벨. Read(SELECT)할 때 공유 잠금을 거는 단계<br/>
```sql
-- T1
set transaction isolation level read committed
begin tran
update ttt set name='sss' where id=1

-- T2
set transaction isolation level read committed
begin tran
select * from ttt;
commit;
```
* T1에서 레코드(id=1)에 X락을 걸고 수정
* T2에서 레코드(id=1)에 S락을 걸려고 하지만 T1에서 X락을 걸어서 액세스 불가, 대기
위의 수순으로 동작하며 Read할 때 S락을 걸기에 타 트랜잭션에서 작업 중인 내용을 조회할 수 없으며 이로 인해 Dirty Read가 방지된다.
<br/><br/>

```sql
-- T1
set transaction isolation level read committed
begin tran
update ttt set name='sss' where id=1

-- T2
set transaction isolation level read committed
begin tran
update ttt set name='nnn' where id=1
```
Write시에 X락을 거는 것은 어느 잠금 레벨에서도 동일하게 잠금으로써 당연히 T2에서 대기하게 된다.
<br/><br/>

```sql
-- T1
set transaction isolation level read committed
begin tran
select * from ttt -- 첫 번째 select
waitfor delay '00:00:10' -- 10초 딜레이
select * from ttt -- 두 번째 select

-- T2
set transaction isolation level read committed
begin tran
update ttt set name='sss' where id=1
commit;
```
"어느 트랜잭션에서 동일한 레코드에 대해 여러번 조회할 때 타 트랜잭션에서 해당 레코드를 변경하면 어떻게되는가"를 묻는 것으로
T1에서 뭔가 락을 걸어서 T2에서 접근을 액세스를 못해서 T1의 두 select 문의 결과가 동일할 것이라고 예상이 되지만 실제로는 그렇지 않다.
그 이유는 아래의 수순으로 진행되기 때문이다.
* T1에서 첫 번째 select문을 진행하기 전에 S락을 건다.
* T1에서 첫 번째 select문이 끝나면 S락을 해제한다.
* T1에서 10초 대기
* T2에서 동일한 레코드에 X락을 걸고 수정 후 해제한다.
* T1에서 동일한 레코드에 다시 S락을 걸고 조회 후 해제한다. (name이 sss로 변경되어 있음)
이와 같이 한 트랜잭션에서 같은 레코드에 여러 번 조회했을 때 그 내용이 동일하지 않은 경우를 Non-Repeatable Read(반복 읽기 불가능)라고 하며 
Read Committed 단계에서는 Dirty Read는 발생하지 않지만 그 외의 문제는 여전히 존재한다.
### Read Committed 정리
Read할 때 S락을 걸기에 더 이상 Dirty Read는 발생하지 않는다. 하지만 SELECT로 읽어 온 후 바로 해제하기 때문에 
다른 문제들(Non-Repeatable Read, Phantom Read)은 여전히 발생한다. <br/>

TODO : <br/>
추가 내용으로 MSSQL에서는 이 단계에서 READ_COMMITTED_SNAPSHOT이라는 옵션의 ON/OFF에 따라 동작이 다르다고 하는데
해당 옵션은 기본적으로 off이며 위의 내용에서 별 다른 내용이 없으며 on을 했을 때 낙관적 락 방식(행 버전 관리)으로 바뀌며 
숙지해야할 내용이 있지만 이는 추후에 기재
## 3-3. Repeatable Read 격리 수준
전 단계(Read Committed)에서 Non-Repeatable read 정합성 문제가 발생한다. 이를 방지하기 위해 추가된 단계로 한 트랜잭션에서
SELECT한 레코드(S락을 걸은)에 대해 그 트랜잭션이 끝날 때까지 S락을 해제하지 않음으로써 Non-Repeatable read를 방지하는,
그러니까 이름 그대로 Repeatable Read가 가능한 단계이다.
```sql
-- T1
set transaction isolation level repeatable read
begin tran
select * from ttt where id=1 -- 1번째 select
waitfor delay '00:00:10'
select * from ttt where id=1 -- 2번째 select
commit
-- T2
set transaction isolation level repeatable read
begin tran
update ttt set name='kkkkk' where id=1
commit
```
* T1에서 레코드(id=1)에 S락을 걸고 읽는다
* T1에서 S락을 해제하지 않고 딜레이
* T2에서 레코드(id=1)에 X락을 걸려고 하지만 S락이 걸려있어 대기
* T1에서 레코드(id=1)을 읽는다. 커밋(이나 롤백으로 트랜잭션 끝낸다.)
* T2에서 X락을 걸고 변경한다.

### Repeatable read 정리
전 단계에서 발생하는 Non-Repeatable Read를 막기 위해서 조금 더 추가된 단계로 해당 문제는 막지만 여전히 Phantom Read에 대한 문제는 남아 있는 단계이다.

## 3-4 Serializable 격리 수준
키 "범위" 잠금을 통해 쿼리 필터 조건을 만족하는 모든 데이터에 대해 논리적인 잠금을 설정한다. 
해석하자면 이미 존재하는 물리적인 데이터 뿐만 아니라,
쿼리 필터 조건을 만족하지만 아직 존재하지 않은 가상의 데이터까지도 잠근다. 이 단계에서 Phantom Read 정합성 문제를 막을 수 있다.

```sql
-- T1
set transaction isolation level serializable
begin tran


-- T2


```

## 3-5 Snapshot
TODO : 추후 수정

## 3-6 트랜잭션 격리 레벨 요약
격리 레벨을 무엇으로 설정하든 Write할 때 X락은 걸린다. 아래의 모든 격리 레벨에 따른 내용 차이들은 S락을 잡는 작업의 변경에 따른 차이이다. 
<table>
          <tr><th>격리 레벨</th><th>설명</th></tr>
          <tr>
                    <td>Read UnCommitted</td>
                    <td>
                              트랜잭션 격리 수준 중 가장 낮은 단계(레벨 0) <br/>
                              SELECT 작업 수행 시 공유 잠금이 걸리지 않기 때문에 트랜잭션이 완료되지 않은 상태도 조회 가능 <br/>
                              대기 없이 빠른 속도로 조회 가능하다는 장점이 있으나 모든 정합성 문제가 다 발생할 수 있다.
                    </td>
          </tr>
          <tr>
                    <td>Read Committed</td>
                    <td>
                              MSSQL 기본 격리 수준 <br/>
                              SELECT할 때 S락이 걸린다. <br/>
                              커밋된 내용만 읽기 때문에 Dirty Read는 발생하지 않으나 그 외의 문제는 여전히 발생한다.
                    </td>
          </tr>
          <tr>
                    <td>Repeatable Read</td>
                    <td>
                              한 트랜잭션에서 걸은 S락이 그 트랜잭션이 끝날 때까지 유지가 되는 단계 <br/>
                              d
                    </td>
          </tr>
          <tr>
                    <td>Serializable</td>
                    <td>d</td>
          </tr>
          <tr>
                    <td>Snapshot</td>
                    <td>d</td>
          </tr>
</table>


출처 : <br/>
https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide?view=sql-server-ver17 <br/>
https://blog.naver.com/ssayagain/90032393567 <br/>
https://jiazzang.tistory.com/13 <br/>
https://battleracoon.tistory.com/2 - sp_lock, 봐야할 거 <br/>
<hr/><br/><br/>


# 그 외 내용
## SP_LOCK, SP_WHO
SP_LOCK, SP_WHO을 사용, 락이 걸린 상황을 체크한다. <br/>
아래 조건 중 하나를 만족하면 해당 세션은 현재 배타 락을 걸고 있는 세션이다. <br/>
* SP_LOCK으로 확인했을 때 배타 잠금(X)이 걸린 세션
* SP_LOCK으로 확인했을 때 blk(Blocked By) 칼럼에 값이 있을 경우, 칼럼값에 해당하는 세션
* SP_WHO로 확인햇을 때 blk 칼럼에 값이 있는 세션

### 테스트1-1
```sql
// DB세션1에서 트랜잭션 걸고 INSERT 
begin tran
insert into ttt(id, name) values(5, 'e');
```
### 테스트1-2
```sql
// DB세션2에서 SELECT
select * from ttt;
```
### 테스트1-3
DB세션1에서 트랜잭션을 걸고 풀지 않아서 DB세션2가 쿼리를 수행하지 못하고 대기하게 된다. <br/>
이 상황에서 SP_LOCK과 SP_WHO를 사용하여 확인한다. <br/>
SP_LOCK <br/>
![sp_lock](https://github.com/user-attachments/assets/2d04129d-0006-40af-b858-192ce44628e9) <br/>
DB세션1(spid-64)이 현재 Mode=X, Status=GRANT로 배타 락을 걸고 있다. <br/>
DB세션2(spid-62)가 현재 Mode=S, Status=WAIT으로 공유 잠금을 걸기 위해 대기중이다. <br/>

SP_WHO <br/>
![sp_who](https://github.com/user-attachments/assets/f611b318-22dc-44db-aaaf-5d69cb23d003) <br/>
DB세션2(spid-62)가 현재 DB세션1(spid-64)에 의해 블록당해 Suspended 상태이다. <br/>
DB세션1(spid-64)이 현재 커맨드 대기중이며 sleep 상태이다. <br/>

### 테스트1-4
현재 락을 점유하고 있는 DB세션1을 kill을 해주거나(Kill 64) 해당 세션에서 commit이나 rollback으로 트랜잭션을
마무리해주면 DB세션2에서 락을 획득하여 처리하고 결과물을 보여준다. <br/>
![rollback](https://github.com/user-attachments/assets/b265332e-4a5a-437f-9600-dff4f306d4fe) <br/>

### 표 해석
자세한 설명은 MSDN에서 참고 <br/>
<table>
          <tr>
                    <th>열 이름</th>
                    <td>spid</td>
                    <td>dbid</td>
                    <td>ObjId</td>
                    <td>IndId</td>
                    <td>Type</td>
                    <td>Resource</td>
                    <td>Mode</td>
                    <td>Status</td>
          </tr>
          <tr>
                    <th>데이터 형식</th>
                    <td>smallint</td>
                    <td>smallint</td>
                    <td>int</td>
                    <td>smallint</td>
                    <td>nchar(4)</td>
                    <td>nchar(32)</td>
                    <td>nvarchar(8)</td>
                    <td>nvarchar(5)</td>
          </tr>
          <tr>
                    <th>설명</th>
                    <td>잠금 요청하는 db 세션 id</td>
                    <td>잠금이 설정된 db의 id <br/> "DB_NAME()" 함수를 사용하여 db를 식별할 수 있다.</td>
                    <td>잠금이 유지되는 개체의 ID 번호이다 <br/> "OBJECT_NAME()" 함수를 사용하여 개체를 식별할 수 있다.</td>
                    <td>잠금이 유지되는 인덱스의 ID 번호</td>
                    <td>
                              잠금 유형<br/>
                              RID:RID(행 식별자), 행 단위 잠금<br/>
                              KEY:인덱스가 있을 때 행 단위 잠금<br/>
                              PAG:데이터 페이지 또는 인덱스 페이지 단위 잠금<br/>
                              EXTENT:인접한 8개의 데이터 페이지 또는 인덱스 페이지 단위<br/>
                              TAB:데이터 및 인덱스를 포함하여 전체 테이블 단위<br/>
                              DB:DB를 잠금<br/>
                              FIL:DB 파일을 잠금
                    </td>
                    <td>
                              잠김 리소스를 식별하는 값 <br/>
                              d
                    </td>
                    <td>
                              잠금 모드 <br/>
                              S:공유 잠금 <br/>
                              
                              X:배타 잠금 <br/>
                              I(Intent):의도적 잠금 <br/>
                              SQL Server가 리소스에 대해 공유 잠금 또는 배타 잠금을 얻으려할 때 먼저 거는 잠금 <br/>
                              테이블 수준으로 걸고, 
                              
                              IX:의도적 배타 잠금 <br/>
                              일단 추후 수정
                              Sch-M(Manipulation):스키마 조작(DDL-Create, Drop 등)시에 스키마에 걸리는 잠금, 모든 잠금에 대해 배타적<br/>
                              Sch-S(Stability):쿼리문 컴파일 시에 발생, Sch-M을 제외한 다른 잠금과 호환 가능 <br/>
                              Range:범위 잠금
                    </td>
                    <td>
                              잠금 요청 상태 <br/>
                              CNVRT:현재 잠금에서 다른 잠금 유형으로 바꾸려고 하지만 다른 곳에서 걸려있는 잠금에 의해 대기중 <br/>
                              GRANT:잠금을 획득한 상태 <br/>
                              WAIT:다른 잠금에 의해 블로킹 당해 대기
                    </td>
          </tr>
</table>

<br/><br/>

## 테스트2
현재 트랜잭션 잠금 레벨 확인하기 <br/>
```sql
SELECT CASE  
          WHEN transaction_isolation_level = 1 
             THEN 'READ UNCOMMITTED' 
          WHEN transaction_isolation_level = 2 
               AND is_read_committed_snapshot_on = 1 
             THEN 'READ COMMITTED SNAPSHOT' 
          WHEN transaction_isolation_level = 2 
               AND is_read_committed_snapshot_on = 0 THEN 'READ COMMITTED' 
          WHEN transaction_isolation_level = 3 
             THEN 'REPEATABLE READ' 
          WHEN transaction_isolation_level = 4 
             THEN 'SERIALIZABLE' 
          WHEN transaction_isolation_level = 5 
             THEN 'SNAPSHOT' 
          ELSE NULL
       END AS TRANSACTION_ISOLATION_LEVEL 
FROM   sys.dm_exec_sessions AS s
       CROSS JOIN sys.databases AS d
WHERE  session_id = @@SPID
  AND  d.database_id = DB_ID();
```

<br/><br/>






