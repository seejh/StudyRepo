출처 : 유튜브 kudvenkat <br/>
https://www.youtube.com/watch?v=OyBwIjnQLtI&list=PL6n9fhu94yhXg5A0Fl3CQAo1PbOcRPjd0 <br/>

# SQL Server가 내부적으로 데이터를 저장하는 실제 방법
알아야 할 기술 용어: 데이터 페이지, 루트 노드, 리프 노드, B 트리, 클러스터형 인덱스 구조 <br/>
<img width="350" height="200" alt="image" src="https://github.com/user-attachments/assets/949ecb58-968c-41e3-8576-781014268ab8" /><br/>

DB에서 데이터는 논리적인 수준에서 행과 열 형식으로 저장되지만 물리적(실제로)으로는 데이터 페이지라는 곳에 저장된다.
데이터 페이지는 SQL Server의 기본 데이터 저장 단위이며 크기는 8KB이다. SQL Server DB 테이블에 데이터를 삽입하면 해당 데이터가
8KB 데이터 페이지에 저장된다. <br/>
<img width="500" height="200" alt="image" src="https://github.com/user-attachments/assets/48d25c6c-fa6e-4937-a5a9-760db76d3c88" /><br/>
데이터들이 8KB 단위의 데이터 페이지로 저장되며 이 데이터 페이지들의 트리 구조로 연결되어 저장된다.<br/>
<img width="500" height="200" alt="image" src="https://github.com/user-attachments/assets/0c07c916-c5c3-4f06-a45b-24f628cb582b" /><br/>

아래와 같이 직원에 대한 정보를 담고 있는 직원 테이블이 있다.<br/>
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/315ba921-17fe-4d06-b174-cd62498fb61b" /><br/>
EmployeeId가 기본키로 클러스터형 인덱스가 있고 데이터 페이지에 있는 실제 데이터들이 EmployeeId 열에 따라 정렬된다.<br/>

실제로 저장되는 형태는 아래와 같다.<br/>
이 트리 형태 구조를 B 트리, 인덱스 B 트리, 클러스터형 인덱스 구조 등으로 부르고 모두 같은 의미를 갖는다.<br/>
<img width="700" height="300" alt="image" src="https://github.com/user-attachments/assets/7755f153-d612-4ec2-b7ef-bd0b1fdff836" /><br/>
### 리프 노드
제일 하단의 노드(Data Rows라 적힌)를 리프 노드 또는 데이터 페이지라고 부르며 이곳에 실제 테이블 데이터가 들어간다. 각 데이터 페이지의 크기는 8KB이며 각 데이터 페이지에 저장되는 데이터량(행의 수)은 실제 저장되는 각 행의 크기에 따라 달라진다.<br/>

### 중간 레벨 노드, 루트 노드
트리의 제일 위에 있는 노드를 루트 노드, 루트 노드와 리프 노드 사이에 있는 노드를 중간 레벨 노드라고 한다.
이 예에서는 쉽게 이해하기 위해 한 계층으로 표현되어 있지만 실제로는 테이블에 있는 데이터의 행 수에 따라 중간 레벨 노드의 개수가 달라지며 이 루트 노드와 중간 레벨 노드에는 인덱스 행이 포함된다.<br/>

인덱스 행에는 주요 인덱스값(여기서는 EmployeeId)과 그에 해당하는 데이터가 실제로 저장되어 있는 곳을 가리키는 포인터(다음 중간 레벨 노드 또는 리프 노드의 데이터 행)가 저장되어 있다. 쉽게 말해 이 노드들에 저장되는 인덱스 행은 해당 인덱스에 해당하는 실제 데이터가 저장하는 공간을 가리키는 포인터들이 저장되어 있다.<br/>
<img width="700" height="300" alt="image" src="https://github.com/user-attachments/assets/13402afe-b189-4c0a-aa4b-51fe313c56d0" /><br/>
쉽게 정리하자면 리프 노드에는 실제 데이터들이 저장되고 중간 노드와 루트 노드에는 인덱스들이 저장된다는 것. <br/>

# SQL Server가 인덱스를 사용하여 탐색하는 방법
EmployeeId=1120인 직원 행을 찾는다고 할 때 "select * from Employees where EmployeeId=1120;"를 수행하여 찾는다.<br/>
<img width="700" height="300" alt="image" src="https://github.com/user-attachments/assets/816452e4-3cb7-428d-b429-7152f9fc7adc" /><br/>
DB 엔진은 루트 노드에서 시작해서 우측 인덱스 노드를 선택한다. DB 엔진은 이 노드가 801에서 1200까지의 EmployeeId를 포함하고 있다는 것을 알고 있기 때문이다. 거기에서 1001에서 1200까지의 직원 데이터 행이 제일 우측 리프 노드에 있기 때문에 제일 우측 리프 노드를 선택한다. 마지막으로 이 리프 노드에서 데이터 행은 EmployeeId를 기준으로 정렬되어 있기 때문에 쉽게 찾을 수 있다.
요점은 데이터를 찾는데 도움이 되는 인덱스가 있다면 SQL Server가 쉽고 빠르게 찾을 수 있다는 것이다. <br/>

# 클러스터형 테이블 vs 힙 테이블
SQL에서 데이터가 저장되는 방식에 따라 테이블이 나눠지고 클러스터형 테이블과 힙 테이블이 있다. 클러스터형 테이블은 클러스터 인덱스를 사용함으로서 해당 테이블의 실제 데이터들이 설정한 클러스터 인덱스 칼럼으로 정렬되는 테이블이고 힙 테이블은 클러스터 인덱스를 설정하지 않아 실제 데이터가 정렬되지 않아 순서에 대한 보장이 되지 않는 테이블이다. <br/>

# 클러스터 인덱스와 논클러스터 인덱스


# SQL Server가 데이터를 찾는 과정을 실제 실행 계획으로 확인
아래 실습을 진행하기 전에 실제 실행 계획을 활성화한다.<br/>

### 실습 환경 구성(테이블 생성, 더미 데이터 삽입)

```sql
-- 테이블 생성
create table employees(
id int primary key identity,
[name] nvarchar(50),
email nvarchar(50),
department nvarchar(50)
)

-- 더미 데이터 백만 건 삽입
set nocount on
declare @counter int = 1

while(@counter <= 1000000)
begin
declare @name nvarchar(50) = 'ABC ' + RTRIM(@counter)
declare @email nvarchar(50) = 'abc ' + RTRIM(@counter) + '@github.com'
declare @dept nvarchar(10) = 'Dept ' + RTRIM(@counter)

insert into employees values(@name, @email, @dept)
set @counter = @counter + 1

-- 10만 건이 삽입될 때마다 출력
if (@counter%100000 = 0)
print RTRIM(@counter) + ' rows inserted'
end
```

### 사용할 수 있는 인덱스가 있는 경우
#### 실행 쿼리
```sql
-- 직원 ID가 932000인 직원 찾기
select * from employees where id=932000;
```

#### 실제 실행 계획
<img width="400" height="500" alt="image" src="https://github.com/user-attachments/assets/5df6ea74-04c2-426c-a0d9-1d15c4c29a20" /><br/>

* Clustered Index Seek 방식
* Number of Rows Read : 1
* Actual Number of Rows for All Executions : 1

현재 EmployeeId가 기본키로 클러스터형 인덱스가 있고 EmployeeId를 탐색하고 있기에 Clustered Index Seek 방식을 사용한다.
Number of Rows Read는 SQL Server가 쿼리 결과를 생성하기 위해 읽어야 하는 행의 수로서 이 예제의 경우 EmployeeId가 고유하고 우리가 찾는 데이터가 EmployeeId 인덱스를 사용하여 특정 행을 직접 읽을 수 있으므로 1 건의 데이터 행만 읽으면 되기에 1로 표기하고
모든 연산의 결과로 가져온 행이 1개 이므로 Actual Number of Rows for All Executions는 1<br/>

### 사용할 수 없는 인덱스가 없는 경우
#### 실행 쿼리
```sql
-- 직원 이름이 ABC 932000인 직원 찾기
select * from Employees where Name='ABC 932000';
```

#### 실제 실행 계획
<img width="400" height="500" alt="image" src="https://github.com/user-attachments/assets/0ed26392-2cdb-42a4-983b-f5df04e6d222" /><br/>

* Clustered Index Scan 방식
* Number of Rows Read : 1000000
* Actual Number of Rows for All Executions : 1

현재 EmployeeId에 클러스터형 인덱스가 걸려있고 직원 이름(Name)와 관련된 인덱스는 없기에 풀 스캔이 발생하게 된다. 이 쿼리를 수행하기 위해 모든 테이블을 다 탐색(풀 스캔)해야 하기에 Number of Rows Read는 100만, 결과로 1 건을 가져오기에 Actual Number of Rows for All Executions는 1<br/>

### 논클러스터 인덱스
클러스터형 인덱스는 1개만 있어야 하고, 위처럼 클러스터형 인덱스가 걸려있는 칼럼 외에 다른 칼럼에 인덱스를 걸어야할 필요가 있을 때
논클러스터 인덱스를 사용할 수 있다. 직원 이름(Name) 칼럼에 논클러스터 인덱스를 추가한다.<br/> 

#### 실행 쿼리
```sql
-- 직원 이름(Name) 칼럼에 논클러스터 인덱스 추가
create nonclustered index idx_name on Employees(Name);

-- 탐색 쿼리 재수행
select * from Employees where Name='ABC 932000';
```

#### 실제 실행 계획
<img width="400" height="500" alt="image" src="https://github.com/user-attachments/assets/eabd476b-ae35-4209-ac0e-5ba5c9ea3515" /><br/>

* Index Seek
* Number of Rows Read : 1
* Actual Number of Rows for All Executions : 1

직원 이름(Name)에 논클러스터 인덱스가 걸려있어 논클러스터 인덱스를 사용하여 탐색한다. <br/>

### 논클러스터 인덱스가 DB에 저장되는 방식
<img width="700" height="400" alt="image" src="https://github.com/user-attachments/assets/ca70fea5-9b31-4176-90f5-9f39f6577f44" /><br/>


논클러스터 인덱스에는 테이블 데이터가 없다. 키와 값(Key Values), 행 로케이터(Row Locators)가 있다.
쉽게 말해 데이터는 없고 데이터들의 위치를 나타내는 포인터들만 있다는 것이고. 

주요 값(이 경우 직원 이름(Name))은 알파벳 순서로 정렬되어 저장
Row Locators에 직원 이름과 행의 클러스터 키가 포함되어 있음



Key Lookup(클러스터형 테이블)과 RID Lookup(힙 테이블) 성능 비교
Estimated Subtree Cost 수치가 같음으로 성능은 비슷.






# 실행 계획 읽는 법
실행 계획을 사용하면 해당 쿼리를 수행하는 과정을 트리 형태로 표현해준다.<br/>
읽는 법은 우측에서 좌측으로, 상단에서 하단 순으로 읽으면 된다. <br/>

## 예시
### 수행 쿼리
<img width="392" height="25" alt="image" src="https://github.com/user-attachments/assets/1b1ef609-ec81-4584-81d2-19d2785cebb4" />

### 실제 실행 계획
<img width="365" height="288" alt="image" src="https://github.com/user-attachments/assets/b2dedd00-2927-4ec8-a08b-483de467c4b1" />

### 실행 계획 트리에서 과정(연산자) 세부 조회
실행 계획에서 한 과정 위에 마우스를 올려 놓으면 아래와 같이 세부적으로 표시됨.<br/>
<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/6f795186-0c7e-4b09-9040-579170b0a8ff" />

<img width="1034" height="500" alt="image" src="https://github.com/user-attachments/assets/34407389-632e-48f8-95d4-d4921ffe2791" /><br/>

### 해석
* 1.Index Seek <br/>

* 2.RID Lookup <br/>

힙 테이블, Name 칼럼에 논클러스터 인덱스 
1 Index Seek
논클러스터 인덱스에서 해당하는 직원 항목 찾음
2 RID Lookup
Row Locators에서 해당 직원의 행 ID 조회
3 Nested Loops
select * from employees where name=''; 쿼리에서 논클러스터 인덱스를 사용할 수 있음 => index seek


## 실행 계획 항목 해석
<table>
  <tr>
    <td>Number of Rows Read</td>
    <td>읽은 전체 행 수</td>
  </tr>
  <tr>
    <td>Actual Number of Rows for all Executions</td>
    <td>쿼리 결과로 반환한 행의 총 수</td>
  </tr>
  <tr>
    <td>Estimated Operator Cost</td>
    <td>
      해당 연산자(Operator) 하나를 수행하는데 드는 비용의 상대 지표.
      내부적으로 Estimated I/O Cost + Estimated CPU Cost를 합쳐서 계산.
    </td>
  </tr>
  <tr>
    <td>Estimated I/O Cost</td>
    <td>
      디스크 I/O 때문에 발생하는 비용의 상대 지표.
      (테이블/인덱스에서 데이터를 읽는 횟수 기반)
      페이지 수, 인덱스 깊이, 예상 Row 수 등을 고려해서 산출.
      디스크가 SSD냐 HDD냐는 직접 반영되지 않고, 기본 통계와 가정치에 기반.
      주로 Table Scan, Index Scan 같은 "많은 데이터를 읽는 연산"에서 높게 나옴.
    </td>
  </tr>
  <tr>
    <td>Estimated CPU Cost</td>
    <td>
      CPU 연산량 때문에 발생하는 비용의 상대 지표.
      (정렬, 조인, 해시 같은 작업량 기반)
      데이터 건수(Row 수) * 연산 복잡도에 비례.
    </td>
  </tr>
  <tr>
    <td>Estimated Subtree Cost</td>
    <td>
      해당 연산자를 포함해 하위 연산자까지 모두 합친 누적 비용의 상대 지표.
      옵티마이저가 해당 연산자와 속하는 모든 하위 연산자 트리를 실행하는데 드는 전체 비용을 보여준다.
    </td>
  </tr>
</table>

_상대 지표?_ <br/>
이 코스트 값이 절대적인 실제 실행 속도(시간)을 의미하는 것이 아니라 "이 경로가 다른 경로보다 더 싸다"를 비교하기 위한 상대 값.
Estimated Subtree Cost=12.34라고 해서 실제로 12초가 걸린다는 의미가 아니라 "다른 경로(Subtree Cost=50)보다 약 4배 유리하다"는 정도로만
해석하면 된다.<br/>


