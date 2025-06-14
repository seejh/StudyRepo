# DB 성능 개선
일단 이 글은 DB 중에서도 MySQL을 기준으로 작성되어 있다. 이 글을 작성한 필자가 MySQL을 기준으로 설명하고 있기 때문이다.

## 1. DB 성능 개선 방법
DB에 부하가 걸렸을 때 성능을 개선하는 방법은 다양하다 주로 아래와 같다.
* SQL 튜닝
* 캐싱 서버 활용(Redis 등)
* 레플리케이션 (Master/Slave)
* 샤딩
* 스케일업(CPU, Memory, SSD 등 하드웨어 업그레이드)

## 2. SQL 튜닝을 먼저 고려해야 한다.
1. SQL 튜닝을 제외한 나머지 방법은 추가적인 시스템을 구축해야 한다. 따라서 금전적, 시간적 비용이 추가적으로 발생한다.
   조금 더 복잡해진 시스템 구조로 인해 관리 비용이 늘어난다. 그에 비해 SQL 튜닝은 기존의 시스템 변경없이 성능을 개선할 수 있다.
2. 근본적인 문제를 해결하는 방법이 SQL 튜닝일 가능성이 높다. SQL 자체가 비효율적으로 작성됐다면 아무리 시스템적으로 성능을 개선한다 하더라도 한계가 있다.
   하지만 SQL 튜닝을 통해 기본적으로 성능을 향상시킨다면 시스템적인 성능 개선이 필요없거나 훨씬 간단한 개선으로 큰 성능 개선 효과를 얻을 수 있다.
   
요약하자면, DB 성능 개선 방법들 중 가장 가성비가 좋고 기본적으로 깔고 가야할 것이 SQL 튜닝이다.

## 3. DB 구조 파악 
어떤 부분에서 DB의 성능을 많이 잡아 먹는지, 어떤 요인이 주로 문제를 일으키는지 파악할 수 있어야 한다.
그러기 위해선 기본적으로 DB의 구조를 알아야 한다. <br/>
아래는 MySQL 아키텍처를 간단히 표현한 것이고 쿼리를 실행하면 어떻게 수행되는지 알아본다. <br/>
![image](https://github.com/user-attachments/assets/d78591d7-6191-4c78-8155-4d310058001b) <br/>
1. 클라이언트가 DB에 SQL 요청을 보낸다.
2. MySQL 엔진에서 옵티마이저가 SQL 문을 분석한 뒤 빠르고 효율적으로 데이터를 가져올 수 있는 계획을 세운다.
   (어떤 순서로 테이블에 접근할 지, 인덱스를 사용할 지, 어떤 인덱스를 사용할 지 등을 결정)
3. 옵티마이저가 세운 계획을 바탕으로 스토리지 엔진에서 데이터를 가져온다.
4. MySQL 엔진에서 정렬, 필터링 등의 마지막 처리를 한 뒤에 클라이언트에게 SQL 결과를 응답한다.

쿼리를 실행하면 위의 순서대로 진행되며 DB 성능 문제의 대부분은 3번 과정, 스토리지 엔진으로부터 데이터를 가져올 때 발생한다.
데이터를 찾기가 어려워서 오래 걸리거나, 가져올 데이터가 너무 많아서 오래 걸린다. SQL 튜닝의 핵심은 스토리지 엔진으로부터
되도록이면 1) 데이터를 찾기 쉽게 바꾸고, 2) 적은 데이터를 가져오도록 바꾸는 것을 말한다.

그럼 이 2가지를 어떻게 해결할 수 있을까
여러가지 방법이 많지만 가장 활용되는 방법이 인덱스 활용이다. 인덱스가 어떤 개념이길래 위 2가지 문제를 해결할 수 있는 지 알아보자.
단순히 인덱스만 적용한다고 해서 무조건 해결되는 것이 아니다. 인덱스를 적절하게 활용해야만 DB 성능이 개선된다. 

## 4. 인덱스
DB 테이블에 대한 검색 성능의 속도를 높여주는 자료 구조, 더 세세하게 설명하자면 아래와 같다.<br/>
특정 컬럼을 기준으로 미리 정렬해놓은 테이블

### 예시를 통해 인덱스의 필요성 이해
<table>
   <tr>
      <th>id(PK)</th><th>이름</th><th>나이</th>
   </tr>
   <tr>
      <td>1</td><td>박미나</td><td>25</td>
   </tr>
   <tr>
      <td>2</td><td>김미현</td><td>23</td>
   </tr>
   <tr>
      <td>3</td><td>마석도</td><td>21</td>
   </tr>
   <tr>
      <td>4</td><td>이재현</td><td>23</td>
   </tr>
   <tr>
      <td>....</td><td>...</td><td>...</td>
   </tr>
   <tr>
      <td>10000</td><td>조민규</td><td>23</td>
   </tr>
</table>
위의 Users 테이블에서 1만 개의 데이터 중에서 나이가 23살인 사용자를 전부 직접 찾으려고 해보자. 나이가 뒤죽박죽 섞여 있기 때문에 모든 행을
일일이 검사해서 23살인 사용자를 뽑아내야 한다. 1만 개의 데이터를 일일이 다 확인해야 하므로 시간이 오래 걸린다. <br/><br/>

하지만 아래와 같이 나이 순으로 정렬된 표가 있다면? <br/>
![image](https://github.com/user-attachments/assets/8750f6bb-d7d5-4ffd-a223-534e9a3add82) <br/>
위의 표에서 ***23살로 시작하는 지점***과 ***24살로 시작하는 지점***만 찾은 뒤 그 사이에 있는 모든 값을 가져오면 된다. <br/>
미리 정렬을 해놓으니 모든 데이터를 일일히 다 확인할 필요가 없어서 훨씬 효율적으로 조회할 수 있다. <br/>

이러한 특징 때문에 데이터를 찾는 속도를 빠르게 만들기 위해서 인덱스를 많이 활용한다. <br/>
위에서 나이 칼럼을 기준으로 정렬된 표가 인덱스이다. 정확히 말하면 user 테이블의 나이 칼럼에 인덱스를 걸어주면 인덱스를 생성한다.
실제 DB에서는 인덱스를 생성한다고해서 실제로 정렬된 표를 확인할 수 없다. 시스템 내부적으로 생성될 뿐이다.


# 여기서부터 수정 필요
# 여기서부터 수정 필요

### 인덱스 실습
인덱스를 배웠고 이것을 실제로 적용해보고 성능이 향상되는 것을 확인하는 것이 중요하다.

1. 테이블과 더미 데이터 생성
```sql
-- 기존 테이블 제거
DROP TABLE IF EXISTS users;

-- 새 테이블 생성
CREATE TABLE users (
id INT AUTO_INCREMENT PRIMARY KEY,
name VARCHAR(100),
age INT
);

-- 더미 생성
-- 기본 제약이 걸려있는 반복 횟수를 뚫어준다. (더미 개수보다 많으면 된다.)
SET SESSION cte_max_recursion_depth = 1000000;

-- 더미 데이터 삽입
INSERT INTO users(name, age)
WITH RECURSIVE cte(n) AS
(
SELECT 1
UNION ALL
SELECT n+1 FROM cte WHERE n<1000000
)
SELECT
-- CONCAT: 매개 변수들을 합쳐서 문자열로 만들어주는 함수
-- LPAD: 왼쪽부터 특정 문자로 특정 자리만큼 채워주는 함수로 오른쪽부터 채우는 건 RPAD
-- 'User' 다음에 7자리 숫자로 구성된 이름 생성
CONCAT('User', LPAD(n, 7, '0')),
-- RAND: 랜덤 숫자 생성 함수, MySQL의 경우 0~1사이의 소수값으로 가져온다.
-- FLOOR: 소수점 버림 함수
-- 1~1000 사이의 랜덤 값으로 나이 생성
FLOOR(1 + RAND() * 1000) AS age
FROM cte;
```

2. 인덱스없이 성능 테스트
```sql
SELECT * FROM users WHERE age=23;
```
위의 쿼리를 실행하여 소요 시간과 행 개수를 체크한다.
테스트는 한 번만 하는 것이 아니라 여러 번 체크해서 샘플을 여러 개를 얻어서 평균값을 확인하고 성능 향상이 몇 %가 향상되었는 지
정확한 수치를 체크해야 한다. <br/>

3. 인덱스 생성 후 확인
```sql
-- users 테이블의 age 칼럼을 기준으로 idx_age라는 이름으로 인덱스 생성
CREATE INDEX idx_age ON users(age);

-- 인덱스 생성 확인, users 테이블의 인덱스 조회
SHOW INDEX FROM users;
```

4. 인덱스 사용 성능 테스트
```sql
SELECT * FROM users WHERE age=23;
```
TODO : 실습해보며 내용 기재해야 한다. <br/>

### 기본으로 설정되는 인덱스(PK)
테이블에서 각 행(레코드)을 식별하기 위한 키를 보고 기본키(Primary Key, PK)라고 하며 대부분의 경우에 테이블을 생성할 때 PK를 설정한다.
PK의 특징 중 하나는 "PK를 기준으로 테이블을 정렬해서 데이터를 보관한다"는 것이다. 

1. 새 테이블 생성 및 테스트 케이스 입력
```sql
-- 기존 테이블 삭제
DROP TABLE IF EXISTS users;

-- 새 테이블 생성
CREATE TABLE users(
id INT PRIMARY KEY, -- PK 설정
name VARCHAR(100)
);

-- 테스트 케이스 입력
INSERT INTO users(id, name) VALUES
(1, 'a'),
(3, 'b'),
(5, 'c'),
(7, 'd');
```
위의 코드로 입력 후 확인하면 1, 3, 5, 7 순으로 데이터가 나온다. <br/>

2. 변경
```sql
UPDATE users
SET id=2
WHERE id=7;
SELECT * FROM users;
```
변경 후 확인하면 4번째 행(id=7)이었던 것이 2번째 행으로 이동해서 1, 2, 3, 5 순으로 되어 있다.
id 칼럼을 기준으로 정렬되었다는 것이고 이유는 PK가 인덱스의 일종이기 때문이다. <br/>

이렇게 원본 데이터 자체가 정렬되는 인덱스를 보고 클러스터링 인덱스라고 부르며 PK만이 유일한 클러스터링 인덱스이다.
일반적으로 인덱스는 원본 테이블과 별개의 정렬된 데이터가 생기는 것인데 클러스터링 인덱스인 PK는 데이터 원본을
정렬시킨다. <br/>

3. 인덱스 조회
```sql
-- users 테이블의 인덱스를 조회
SHOW INDEX FROM users;
```

### 제약 조건을 추가하면 자동으로 생성되는 인덱스 (UNIQUE)
MySQL에서 UNIQUE라는 제약 조건을 추가하면 자동으로 인덱스가 생성된다.
MySQL에서 UNIQUE라는 제약 조건을 사용할 때 기본적으로 인덱스의 원리를 사용해서 UNIQUE 제약 조건을 걸기 때문에 인덱스가 자동으로
생성된다.

1. 테이블 새로 생성 및 유니크 제약 조건 설정
```sql
DROP TABLE IF EXISTS users;

CREATE TABLE users (
id INT AUTO_INCREMENT PRIMARY KEY,
name VARCHAR(100) UNIQUE -- 유니크 제약 조건

-- 인덱스 생성되었는지 확인
SHOW INDEX FROM users;
);
```
위의 코드를 실행하면 테이블이 생성되고 id, name 필드 기반의 index들이 생긴다.

### 인덱스 사용 시 주의점
인덱스를 사용하면 읽기(Read-SELECT) 성능이 향상되지만 쓰기(Write-INSERT, UPDATE, DELETE) 성능은 하락한다. <br/>
![image](https://github.com/user-attachments/assets/6b20df42-e6aa-443f-b342-63df731c5bde) <br/>
인덱스를 추가한다는 건 위의 그림처럼 인덱스용 테이블이 추가적으로 생성된다는 뜻이다. 인덱스가 없으면 원본 테이블에만
데이터를 추가하면 되지만 인덱스를 사용하고 있으면 원본 테이블과 인덱스용 테이블 둘 다에 데이터를 넣어야 하기 때문에
더 느리다. 이러한 이유로 인덱스의 개수가 많아지면 많아질수록 성능은 느려질 수 밖에 없다.

#### 인덱스로 인해 느리다는 것을 이론적으로는 이해했다. 실제로 확인해보자.
인덱스가 없는 테이블과 인덱스가 많이 걸린 테이블 간의 쓰기(INSERT) 성능 비교 <br/>

1. 테스트 환경 구축(인덱스가 없는 테이블, 인덱스가 있는 테이블 생성)
```sql
-- 인덱스 없는 테이블
CREATE TABLE TableNoIndex (
id INT AUTO_INCREMENT PRIMARY KEY,
col1 INT, col2 INT, col3 INT, col4 INT, col5 INT, col6 INT, col7 INT, col8 INT, col9 INT, col10 INT
);

-- 인덱스 있는 테이블
CREATE TABLE TableManyIndex (
id INT AUTO_INCREMENT PRIMARY KEY,
col1 INT, col2 INT, col3 INT, col4 INT, col5 INT, col6 INT, col7 INT, col8 INT, col9 INT, col10 INT

-- 인덱스 있는 테이블의 모든 필드를 인덱스 등록
CREATE INDEX idx_col1 ON TableManyIndex(col1);
CREATE INDEX idx_col2 ON TableManyIndex(col2);
CREATE INDEX idx_col3 ON TableManyIndex(col3);
CREATE INDEX idx_col4 ON TableManyIndex(col4);
CREATE INDEX idx_col5 ON TableManyIndex(col5);
CREATE INDEX idx_col6 ON TableManyIndex(col6);
CREATE INDEX idx_col7 ON TableManyIndex(col7);
CREATE INDEX idx_col8 ON TableManyIndex(col8);
CREATE INDEX idx_col9 ON TableManyIndex(col9);
CREATE INDEX idx_col10 ON TableManyIndex(col10);
);
```

2. 쓰기(insert) 성능 비교
```sql
-- 높은 반복 횟수 설정
SET SESSION cte_max_recursion_depth = 100000;

-- 인덱스가 없는 테이블에 데이터 10만 개 삽입
INSERT INTO TableNoIndex (col1, col2, col3, col4, col5, col6, col7, col8, col9, col10)
WITH RECURSIVE cte AS
(
SELECT 1 AS n
UNION ALL
SELECT n+1 FROM cte WHERE n<100000
)
SELECT
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000)
FROM cte;

-- 인덱스가 많이 걸린 테이블에 데이터 10만 개 삽입
INSERT INTO TableManyIndex (col1, col2, col3, col4, col5, col6, col7, col8, col9, col10)
WITH RECURSION cte AS
(
SELECT 1 AS n
UNION ALL
SELECT n+1 FROM cte WHERE n<100000
)
SELECT
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000),
FLOOR(RAND() * 1000)
FROM cte;
```
위의 코드로 10번 가량 실습을 진행한다고 하면 TableNoIndex의 경우 평균적으로 320ms가 걸리고
TableManyIndex의 경우
* 첫 번째 : 1900ms
* 두 번재(10만 건 누적되어있음) : 2500ms
* 세 번째(20만 건 누적되어있음) : 2660ms
* 네 번째(30만 건 누적되어있음) : 3000ms
* 다섯 번째(40만 건 누적되어있음) : 3200ms
이처럼 데이터의 규모가 커지면 커질수록 속도가 느려지는 모습을 보이기에 인덱스를 사용할 때에는
인덱스를 사용하면 읽기 성능은 상승하나 쓰기 성능은 하락한다는 점을 유의하고 인덱스의 사용을 최소화하려고 노력해야한다.

## 5. SQL 실행 계획 사용
실행 계획이란 DB 옵티마이저가 SQL문을 어떤 방식으로 처리할 지를 계획한 것을 의미한다. 앞의 3. DB 구조 파악에서 설명한 내용으로
우리가 쿼리를 요청하면 DB 엔진의 옵티마이저로 실행 계획을 세우고 그 실행 계획에 따라 스토리지 엔진에서 데이터를 가져온다고 했다. <br/>

이 실행 계획을 보고 비효율적인 부분이 있는지 확인하고 비효율적인 부분이 있다면 DB 옵티마이저가 효율적인 방법을 계획할 수 있도록
튜닝하는 것이 이 장의 목표이다. <br/>

### 실행 계획 확인하는 법
```sql
-- 실행 계획 조회
EXPLAIN SQL문

-- 실행 계획 조회(자세히)
EXPLAIN ANALYZE SQL문
```

### 실행 계획 실습
```sql
-- 기존 테이블 제거 및 새 테스트 테이블 생성
DROP TABLE IF EXISTS users;

CREATE TABLE users(
id INT AUTO_INCREMENT PRIMARY KEY,
name VARCHAR(100),
age INT
);

-- 테스트 데이터 삽입
INSERT INTO users(name, age) VALUES
('박미나', 26),
('김미현', 23),
('김민재', 21),
('이재현', 24),
('조민규', 23),
('하재원', 22),
('최지우', 22);

-- 실행 계획 조회
EXPLAIN SELECT * FROM users
WHERE age = 23;
```
EXPLAIN SELECT * FROM users WHERE age=23 쿼리를 실행하면 아래와 같이 뜬다. <br/>
![image](https://github.com/user-attachments/assets/cd363404-62d7-495e-a80a-f4ddbdcfa985) <br/>
일단 굵직한 거 위주로 짚고 넘어간다. <br/>
* id : 실행 순서
* table : 조회한 테이블 명
* type : 테이블의 데이터를 어떤 방식으로 조회했는지
* possible_keys : 사용할 수 있는 인덱스 후보군
* key : 사용한 인덱스
* ref : 테이블을 조인하는 상황에서 어떤 값을 기준으로 조인을 했는지
* rows : 이 쿼리를 수행하는데 액세스한 데이터 수
* filtered : 필터 조건(where문 등)으로 데이터를 얼마나 줄였는지
이 값이 30이라면 100개의 데이터를 가지고와서 70개를 버리고 30개의 데이터만 응답을 한 것을 의미하며
이 값이 낮다는 것은 쓸모없는 데이터를 많이 가지고 왔다는 의미라고 한다. <br/>

데이터를 찾을 때 데이터가 많을 수록 오래걸린다. SQL 튜닝의 핵심은 rows의 값을 어떻게 줄이는가가 핵심이다. <br/>
rows와 filtered의 값은 정확한 값이 아니라 추정치라고 한다. <br/>

다음은 실행 계획에 대한 자세한 정보 조회, EXPLAIN ANALYZE이다. <br/>
![image](https://github.com/user-attachments/assets/5e694e73-b6db-4acb-8281-be6065177783) <br/>
실행되는 단계별로 나눠서 표현하고 있으며 아래부터 시작으로 아래에서 위로 읽으면 되며 해석은 아래처럼 하면 된다.
```sql
Table scan on users (cost=0.95 rows=7) (actual time=0.0563..0.0673 rows=7 loop=1)
```
users 테이블에 대해 테이블 스캔(풀 스캔)을 진행했으며 0.0673ms가 걸렸고 이 작업으로 7행을 액세스하여 조회해서 가져왔다. <br/>
```sql
Filter:(users.age=23) (cost=0.95 rows=1) (actual time=0.0646..0.0748 rows=2 loop=1)
```
위에서 조회해온 데이터에 필터링(users.age=23) 작업을 했는데 0.0075ms(0.0748 - 0.0673) 가량 걸렸고 2개 행만 남기고 걸러냈다. <br/>

### 실행 계획에서 type 의미 분석하기(ALL, index)
#### 1) ALL : 풀 테이블 스캔 (Full Table Scan)
인덱스를 활용하지 않고 테이블을 처음부터 끝까지 전부 다 뒤져서 데이터를 찾는 방식. <br/>
처음부터 끝까지 전부 다 뒤져서 데이터를 찾는 방식이다보니 비효율적이다. <br/>
<img src="https://github.com/user-attachments/assets/18002d54-93f6-4b8e-bd83-dda426dd0bfa" width="600" height="400" />

##### ALL : 풀 테이블 스캔 예제
```sql
CREATE TABLE users(
id INT AUTO_INCREMENT PRIMARY KEY,
name VARCHAR(100),
age INT
);

INSERT INTO users(name, age) VALUES
('Alice', 30),
('Bob', 23),
('Charlie', 35);

EXPLAIN SELECT * FROM users WHERE age=23;
```
위의 코드를 실행하면 아래와 같이 표가 나온다. <br/>
![image](https://github.com/user-attachments/assets/70a0c7d7-13ab-41cd-95f5-2c9f7b85b80b) <br/>
type에 ALL이라고 표시되며 풀 테이블 스캔을 했다고 알려준다. "왜 풀 테이블 스캔을 했는가"를 살펴보면 users 테이블은
현재 id 칼럼에 인덱스(PK)가 걸려있고 SELECT는 인덱스가 걸려있지 않은 age 칼럼을 조건으로 조회하므로 풀 테이블 스캔을 하게 된다.

#### 2) index : 풀 인덱스 스캔(Full Index Scan)
인덱스 테이블을 처음부터 끝까지 다 뒤져서 데이터를 찾는 방식. <br/>
인덱스 테이블은 실제 테이블보다 크기가 훨씬 작기 때문에 풀 테이블 스캔보다야 효율적이지만 이것도 인덱스 테이블 전체를 읽어야 하기 때문에
아주 효율적이라고 볼 수는 없다. <br/>
<img src="https://github.com/user-attachments/assets/6167a7a6-61f2-448d-b699-265b52b2c964" width="600" height="400" />

##### index : 풀 인덱스 스캔 예제
```sql
SET SESSION cte_max_recursion_depth=1000000;

-- 더미 데이터 삽입
INSERT INTO users(name, age)
WITH RECURSIVE cte(n) AS
(
SELECT 1
UNION ALL
SELECT n+1 FROM cte WHERE n<1000000
)
SELECT
CONCAT('User', LPAD(n, 7, '0')),
FLOOR(1 + RAND() * 1000) AS age
FROM cte;

-- 위의 코드까지 진행하여 테이블에 일단 더미 데이터 100만 건 삽입 후 아래 코드로 인덱스 생성
-- users 테이블의 name 칼럼 인덱스 걸어준다.
CREATE INDEX idx_name ON users(name);

-- 실행 계획 확인(SELECT)
EXPLAIN SELECT * FROM users
ORDER BY name
LIMIT 10;
```
위의 예제 코드로 더미 데이터 삽입, 인덱스 생성 후 아래의 실행 계획을 확인하면 아래와 같이 나온다. <br/>
![image](https://github.com/user-attachments/assets/61040451-2399-42f7-b77d-fe4aa1cf812e) <br/>
type에 index로 인덱스 풀 스캔을 했다고 알려준다. <br/>
"SELECT * FROM users ORDER BY name LIMIT 10;", 이 구문은 users 테이블에서 name을 기준으로 정렬을 시켜서 상위 10개만 가져오라는 코드로
name은 인덱스가 걸려있고 이미 정렬되어 있으므로 따로 할 필요가 없어서 빠르게 실행이 가능하다. 아래와 같이 동작한다. <br/>
1) name을 기준으로 정렬해서 데이터를 가져와야 하기 때문에, name을 기준으로 정렬되어 있는 인덱스를 조회한다.
   (덩치가 큰 users 테이블의 데이터를 하나씩 찾아보면서 정리를 하는 것보다, 이미 name을 기준으로 정렬되어 있는 인덱스를 참고하는 게 효율적이라고 판단한 것이다.)
2) 모든 인덱스의 값을 다 불러온 뒤에 최상단 10개의 인덱스만 뽑아낸다.
3) 10개의 인덱스에 해당하는 데이터를 users 테이블에서 조회한다.

#### 3) const : 1건의 데이터를 바로 찾을 수 있는 경우
조회하고자 하는 1건의 데이터를 헤매지 않고 바로 액세스해서 찾아올 수 있을 때 const가 출력된다. 고유 인덱스 또는 기본 키를 사용해서
1건의 데이터만 조회한 경우에 const가 출력된다. 가장 효율적인 방법이다. 
정리하자면 2가지 조건이 붙는다.
1) 인덱스 사용
   인덱스를 사용하지 않으면 특정 값을 일일히 다 뒤저야 한다. 그래서 바로 접근해서 찾을 수가 없다.
2) 고유해야 한다(UNIQUE)
   인덱스가 있는데 고유하지 않다면 원하는 데이터를 찾았다고 하더라도, 나머지 데이터에 같은 값이 있을 지도 모르기에 다른 데이터들도
   체크해야 한다.
   고유 인덱스와 기본 키는 전부 UNIQUE한 특성을 가지고 있다.
<img src="https://github.com/user-attachments/assets/12235d58-63ed-41c8-b28c-77cf04d60b06" width="600" height="400" />

##### const : 1건의 데이터 조회 예제
```sql
DROP TABLE IF EXISTS users;

CREATE TABLE users(
id INT AUTO_INCREMENT PRIMARY KEY,
account VARCHAR(100) UNIQUE
);

INSERT INTO users(account) VALUES
('user1@email.com'),
('user2@email.com'),
('user3@email.com');

EXPLAIN SELECT * FROM users WHERE id=3; -- 쿼리1
EXPLAIN SELECT * FROM users WHERE account='user3@example.com'; -- 쿼리2
```
###### 쿼리1, 2 실행 결과
![image](https://github.com/user-attachments/assets/a69c6801-8351-4aa9-bc9a-cc907d882b83) <br/>
![image](https://github.com/user-attachments/assets/5486efb3-29d0-4b3e-8de8-db96589e686e) <br/>
id와 account 둘 다 unique한 인덱스가 걸려있기 때문에 위와 같은 조건에서 빠르게 찾아온다.

#### 4) range : 인덱스 레인지 스캔 (Index Range Scan)
인덱스의 범위를 조회하는 경우를 말한다. 범위란 BETWEEN, 부등호(<, >, <=, >=), IN, LIKE를 활용한 데이터 조회를 뜻한다.
이 방식도 인덱스를 활용하기 때문에 효율적인 방식이다. 하지만 인덱스를 사용하더라도 데이터를 조회하는 범위가 클 경우 성능 저하의
원인이 되기도 한다. <br/>
<img src="https://github.com/user-attachments/assets/445f7bbe-6da2-4fec-bbe2-2f2452c57c34" width="600" height="400" />

##### range : 인덱스 레인지 스캔 예제
```sql
-- 100만 건
SET SESSION cte_max_recursion_depth = 1000000;

-- 더미 삽입
INSERT INTO users(age)
WITH RECURSIVE cte(n) AS
(
SELECT 1
UNION ALL
SELECT n+1 FROM cte WHERE n<1000000
)
SELECT
FLOOR(1 + RAND() * 1000) AS age
FROM cte;

-- 인덱스 생성(age)
CREATE INDEX idx_age ON users(age);

-- 테스트
EXPLAIN SELECT * FROM users
WHERE age BETWEEN 10 and 20;

EXPLAIN SELECT * FROM users
WHERE age IN (10, 20, 30);

EXPLAIN SELECT * FROM users
WHERE age < 20;
```
위의 테스트를 진행하면 아래와 같이 type에 range로 나오며 index 범위 기반으로 탐색을 했다는 것을 알려준다. <br/>
![image](https://github.com/user-attachments/assets/459d549f-843a-4bce-a1bc-da7bb8332b81) <br/>

![image](https://github.com/user-attachments/assets/b6f94644-aaf9-46d3-a475-a0e5d41d056a) <br/>
WHERE 문의 부등호(>, <, <=, >=, =), IN, BETWEEN, LIKE와 같은 곳에서 사용되는 칼럼은 인덱스를 설정하면 성능이 향상될 가능성이 높다.

#### 5) ref : 비고유 인덱스를 활용하는 경우
UNIQUE하지 않은 인덱스를 사용하는 경우 type에 ref가 출력된다.

##### ref : 비고유 인덱스 예제
```sql
-- 테이블 생성
CREATE TABLE users(
id INT AUTO_INCREMENT PRIMARY KEY,
name VARCHAR(100)
);

-- 테스트 데이터 삽입
INSERT INTO users(name) VALUES
('박재성'),
('김지현'),
('이지훈');

-- 인덱스 생성 (name)
CREATE INDEX idx_name ON users(name);

-- name을 조건으로 조회
EXPLAIN SELECT * FROM users WHERE name='박재성';
```
위의 코드를 실행하면 아래와 같이 나온다. <br/>
만약 name 칼럼이 UNIQUE 했으면 const라고 나왔겠지만 비고유 인덱스라 ref라 나왔다. <br/>
![image](https://github.com/user-attachments/assets/9df04ff2-edea-46d6-b8b9-e7ca8db78e6b) <br/>


eq_ref, index_merge, ref_or_null 등 다양한 타입들이 존재한다. 하지만 모든 타입들을 다 미리 공부할 필요는 없다.
자주 나오는 type에 먼저 익숙해진 다음에 더 깊이 공부하고 싶을 때 다른 type들도 찾아서 공부하면 된다.

## 6. SQL 튜닝
### 6-1. 한 번에 너무 많은 데이터를 조회하는 SQL문 튜닝
```sql
DROP TABLE IF EXISTS users;

CREATE TABLE users(
id INT AUTO_INCREMENT PRIMARY KEY,
name VARCHAR(100),
age INT
);

-- 테스트 데이터 100만 건 삽입
INSERT INTO users(name, age)
WITH RECURSIVE cte(n) AS
(
SELECT 1
UNION ALL
SELECT n+1 FROM cte WHERE n<1000000
)
SELECT
CONCAT('User', LPAD(n, 7, '0')),
FLOOR(1 + RAND() * 1000) AS age
FROM cte;

-- 만 건 조회 테스트
-- mysql 워크벤치, dbeaver 같은 GUI 툴은 조회할 수 있는 데이터 수의 리밋을 걸어놓는다.
-- 혹시나 조회가 잘 안되면 해당 리밋을 풀어주고 사용해야 한다.
SELECT * FROM users LIMIT 10000;

SELECT * FROM users LIMIT 10;
```
위의 예제를 실행하면 10000 건을 찾아오는 것은 평균 200ms, 10 건 찾아오는 것은 평균 20ms 걸린다.
위의 내용으로 알 수 있는 것은 아래와 같다.
* 데이터가 많으면 많을수록 느려진다.
* 조회 결과 데이터의 개수를 최소화해야 한다.
실제 페이스북, 인스타그램의 서비스를 보더라도 한 번에 모든 게시글의 데이터를 불러오지 않는다. 스크롤을 내리면서
필요한 데이터를 그때그때 로딩하는 방식이다. 다른 커뮤니티 서비스의 게시판을 보면 페이지네이션을 적용시켜서 일부
데이터만 조회하려고 한다. 그 이유가 조회하는 데이터의 개수가 성능에 많은 영향을 끼치기 때문이다. <br/>

데이터를 조회할 때 한 번에 너무 많은 데이터를 조회하는 것은 아닌 지 체크하고 LIMIT, WHERE 문 등을 활용해서
SQL문을 통해 조회해오는 데이터의 수를 줄이는 것을 고려하는 것이 좋다.

### 6-2. WHERE 문이 사용된 SQL문 튜닝하기
#### 최근 3일 이내에 가입한 유저 조회하기
```sql
CREATE TABLE users(
id INT AUTO_INCREMENT PRIMARY KEY,
name VARCHAR(100),
department VARCHAR(100),
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

SET SESSION cte_max_recursion_depth = 1000000;

INSERT INTO users(name, department, created_at)
WITH RECURSIVE cte(n) AS
(
SELECT 1
UNION ALL
SELECT n+1 FROM cte WHERE n<1000000
)
SELECT
CONCAT('User', LPAD(n, 7, '0')) AS name,
CASE
WHEN n%10=1 THEN 'Engineering'
WHEN n%10=2 THEN 'Marketing'
WHEN n%10=3 THEN 'Sales'
WHEN n%10=4 THEN 'Finance'
WHEN n%10=5 THEN 'HR'
WHEN n%10=6 THEN 'Operations'
WHEN n%10=7 THEN 'IT'
WHEN n%10=8 THEN 'Customer Service'
WHEN n%10=9 THEN 'Research and Development'
ELSE 'Product Management'
END AS department, -- 의미 있는 단어 조합으로 부서 이름 생성

-- 최근 10년 내 임의의 날짜 생성
TIMESTAMP(DATE_SUB(NOW(), INTERVAL FLOOR(RAND() * 3650)DAY) + INTERVAL FLOOR(RAND() * 86400)SECOND) AS created_at
FROM cte;

-- 확인
SELECT COUNT(*) FROM users;
SELECT * FROM users LIMIT 10;

-- 성능 측정 : create_at이 3 이하인 데이터 조회
-- 튜닝 전 쿼리
EXPLAIN SELECT * FROM users
WHERE created_at >= DATE_SUB(NOW(), INTERVAL 3 DAY);

-- 튜닝 후 쿼리
CREATE INDEX idx_created_at ON users(created_at);

SHOW INDEX FROM users;

EXPLAIN SELECT * FROM users
WHERE created_at >= DATE_SUB(NOW(), INTERVAL 3 DAY);
```
통상 250ms 소요, ALL(풀 테이블 스캔), Rows:99만
통상 50ms 소요, range(인덱스 레인지 스캔), Rows:1147

#### Sales 부서이면서 최근 3일 이내에 가입한 유저 조회하기
앞 전의 예에서 조건이 추가됨 (Sale부서) <br/>
```sql
-- 튜닝 전 : department가 Sales이고 created_at이 3 이하인 데이터
EXPLAIN ANALYZE SELECT * FROM users
WHERE department='Sales'
AND created_at >= DATE_SUB(NOW(), INTERVAL 3 DAY)

-- 튜닝
SELECT * FROM users
WHERE department='Sales'
AND created_at >= DATE_SUB(NOW(), INTERVAL 3 DAY);

-- 1. created_at에 index 설정
CREATE INDEX idx_created_at ON users(created_at);
30ms 소요

-- 2. department에 index 설정
ALTER TABLE users DROP INDEX idx_created_at;
CREATE INDEX idx_department ON users(department);
140ms 소요

-- 3. 둘 다 설정


```
통상 200ms 소요, ALL(풀 테이블 스캔), Rows:127 <br/>
![image](https://github.com/user-attachments/assets/efb18af8-925a-4c6f-80bc-21c4f0456745) <br/>
탐색 : ALL(풀 테이블 스캔) 수행, Rows:1e+6(10^6, 100만 건) 건 액세스하여 읽어옴, 152ms 소요 + <br/>
필터링 : 가져온 데이터에서 department가 sales인 데이터를 가져온다
Rows:127, 약 50ms 소요(204 - 152)

통상 

출처 : <br/>
https://www.youtube.com/watch?v=vbatA68GL1I&list=PLtUgHNmvcs6rJBDOBnkDlmMFkLf-4XVl3&index=4 <br/>
<hr/><br/><br/>








