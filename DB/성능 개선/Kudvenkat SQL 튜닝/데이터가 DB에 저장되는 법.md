출처 : 유튜브 kudvenkat <br/>
https://www.youtube.com/watch?v=OyBwIjnQLtI&list=PL6n9fhu94yhXg5A0Fl3CQAo1PbOcRPjd0 <br/>

# SQL Server가 내부적으로 데이터를 저장하는 방법
알아야 할 기술 용어: 데이터 페이지, 루트 노드, 리프 노드, B 트리, 클러스터형 인덱스 구조 <br/>
<img width="350" height="200" alt="image" src="https://github.com/user-attachments/assets/949ecb58-968c-41e3-8576-781014268ab8" /><br/>

DB에서 데이터는 논리적인 수준에서 행과 열 형식으로 저장되지만 물리적(실제로)으로는 데이터 페이지라는 곳에 저장된다.
데이터 페이지는 SQL Server의 기본 데이터 저장 단위이며 크기는 8KB이다. SQL Server DB 테이블에 데이터를 삽입하면 해당 데이터가
8KB 데이터 페이지에 저장된다. <br/>
<img width="500" height="200" alt="image" src="https://github.com/user-attachments/assets/48d25c6c-fa6e-4937-a5a9-760db76d3c88" /><br/>
SQL Server에서 실제로 해당 데이터 페이지들이 연결된 트리 구조로 저장된다.<br/>
<img width="500" height="200" alt="image" src="https://github.com/user-attachments/assets/0c07c916-c5c3-4f06-a45b-24f628cb582b" /><br/>

예시를 통해서 알아본다. 직원에 대한 정보를 담고 있는 직원 테이블이 있다.<br/>
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/315ba921-17fe-4d06-b174-cd62498fb61b" /><br/>
EmployeeId가 기본 키이다. 그 말인 즉슨 이 EmployeeId 열에 클러스터형 인덱스가 생성되고 데이터들이 물리적으로 EmployeeId 열에 따라 정렬된다는 것이다. 이 데이터들은 데이터 페이지에 저장된다.<br/>

<img width="700" height="300" alt="image" src="https://github.com/user-attachments/assets/7755f153-d612-4ec2-b7ef-bd0b1fdff836" /><br/>
위와 같은 트리 형태 구조를 B트리, 인덱스 B트리, 클러스터형 인덱스 구조 등으로 부르고 모두 같은 의미를 갖는다.<br/>
### 리프 노드
위 그림에서 제일 아래에 보이는 노드(Data Rows라 적힌)로, 리프 노드 또는 데이터 페이지라고 부르며 이 리프 노드에 테이블 데이터가 들어간다. 각 데이터 페이지의 크기는 8KB이며 각 데이터 페이지에 저장되는 행의 수는 실제 저장되는 각 행의 크기에 따라 달라진다.

### 중간 레벨 노드, 루트 노드
트리의 제일 위에 있는 노드를 루트 노드, 루트 노드와 리프 노드 사이에 있는 노드를 중간 레벨 노드라고 한다.
이 예에서는 쉽게 이해하기 위해 한 계층으로 표현했지만 실제로는 테이블에 있는 데이터의 행 수에 따라 중간 레벨 노드의 개수가 달라진다고 하며 이 루트 노드와 중간 레벨 노드에는 인덱스 행이 포함된다.
각 인덱스 행에는 키 값(여기서는 EmployeeId)과 중간 레벨 노드 또는 리프 노드의 데이터 행을 가리키는 포인터가 포함된다.
쉽게 말해 이 노드들에는 DB 엔진이 데이터를 빠르게 찾는데 도움이 되는 일련의 포인터가 있다. <br/>
<img width="700" height="300" alt="image" src="https://github.com/user-attachments/assets/13402afe-b189-4c0a-aa4b-51fe313c56d0" /><br/>

이 또한 예로 이해를 해보자. EmployeeId=1120인 직원 행을 찾는다고 한다.<br/>
"select * from Employees where EmployeeId=1120;"과 같은 쿼리를 수행하여 찾는다.<br/>
<img width="700" height="300" alt="image" src="https://github.com/user-attachments/assets/816452e4-3cb7-428d-b429-7152f9fc7adc" /><br/>
DB 엔진은 루트 노드에서 시작해서 우측 인덱스 노드를 선택한다. DB 엔진은 이 노드가 801에서 1200까지의 EmployeeId를 포함하고 있다는 것을 알고 있기 때문이다. 거기에서 1001에서 1200까지의 직원 데이터 행이 제일 우측 리프 노드에 있기 때문에 제일 우측 리프 노드를 선택한다. 마지막으로 이 리프 노드에서 데이터 행은 EmployeeId를 기준으로 정렬되어 있기 때문에 쉽게 찾을 수 있다.
요점은 데이터를 찾는데 도움이 되는 인덱스가 있다면 SQL Server가 쉽고 빠르게 찾을 수 있다는 것이다. <br/>

이 예에서는 EmployeeId 열에 클러스터형 인덱스가 있고 덕분에 EmployeeId로 검색하면 SQL Server는 쉽고 빠르게 데이터를 찾을 수 있지만 직원의 이름(Name)으로 검색하면 Name에는 인덱스가 걸려있지 않기 때문에 데이터를 쉽게 찾을 수 있는 방법이 없기에 테이블의 모든 레코드를 읽는 테이블 풀 스캔이 발생하게 되고 이는 성능 관점에서 볼 때 매우 비효율적이다. 이 때 직원 이름 열에 논클러스터형 인덱스를 생성하게 된다. <br/>

<hr/><br/><br/>

# SQL 인덱스가 사용되는 법
인덱스가 사용되는 방법을 실습을 통해 확인한다. 해당 예제들을 실행하기 전에 실제 실행 계획을 활성화한 후 진행한다.

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

EmployeeId는 기본키로 클러스터형 인덱스이기에 Clustered Index Seek 방식을 사용하여 탐색한다.
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

찾는 것이 직원 이름(Name)이고 이름에 관련된 인덱스는 없기에 풀 스캔이 발생하게 된다. 그래서 Number of Rows Read는 풀스캔하여 100만, 해당 쿼리의 결과로 1 행을 가져온다. <br/>

### 논클러스터 인덱스
클러스터형 인덱스는 1개만 있어야 하기에 위와 같은 경우에 인덱스를 더 추가하고자 하면 논클러스터 인덱스를 추가할 수 있다.
이 예의 경우에는 직원 이름(Name)에 추가할 수 있다. <br/>

#### 실행 쿼리
```sql
-- 논클러스터 인덱스 추가(직원 테이블의 Name 칼럼 인덱스 추가)
create nonclustered index idx_name on Employees(Name);

-- 다시 탐색 쿼리 수행
select * from Employees where Name='ABC 932000';
```

#### 실제 실행 계획
<img width="400" height="500" alt="image" src="https://github.com/user-attachments/assets/eabd476b-ae35-4209-ac0e-5ba5c9ea3515" /><br/>

* 

### 논클러스터 인덱스가 DB에 저장되는 방식
<img width="700" height="400" alt="image" src="https://github.com/user-attachments/assets/ca70fea5-9b31-4176-90f5-9f39f6577f44" /><br/>


논클러스터 인덱스에는 테이블 데이터가 없다. 키와 값(Key Values), 행 로케이터(Row Locators)가 있다.
쉽게 말해 데이터는 없고 데이터들의 위치를 나타내는 포인터들만 있다는 것이고. 

주요 값(이 경우 직원 이름(Name))은 알파벳 순서로 정렬되어 저장
Row Locators에 직원 이름과 행의 클러스터 키가 포함되어 있음






# 실행 계획 읽는 법
Number of Rows Read
읽은 전체 행 수
Actual Number of Rows for All Executions
결과로 반환한 행의 총 수

예시를 통한 이해
예를 들어, 100만 건의 행을 가진 테이블에서 where 절을 사용하여 특정 조건을 만족하는 1천 개의 행을 조회한다고 할 때,
Number of Rows Read는 인덱스가 없거나 비효율적인 인덱스를 사용할 경우, SQL Server는 테이블 풀 스캔을 하여 100만 건을 읽어서 이 값이 100만으로 표시될 수 있다.
Actual Number of Rows for All Executions는 실제로 조건을 만족하여 반환되는 행의 수로 1000개가 표시된다.


