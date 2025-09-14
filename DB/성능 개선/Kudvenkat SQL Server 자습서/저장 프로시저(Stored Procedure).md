# 저장 프로시저(Stored Procedure)
임시 SQL이나 인라인 SQL보다 저장 프로시저를 사용하는 것의 장점

## 쓰는 이유, 장점
### 1. 실행 계획 재사용 가능
일반적으로 쿼리를 수행하면 아래와 같은 과정을 거친다.
* 구문 확인
* 쿼리 컴파일
* 실행 계획 생성
* 실행

매번 이것을 반복하는데 저장 프로시저를 사용하면 실행 계획이 SQL Server에 캐싱되어 재사용할 수 있다.
요즘은(SQL Server 버전이 최근의 경우) 임시 쿼리를 사용하더라도 SQL Server가 실행 계획을 캐싱하여 재사용할 수 있지만
이 경우, 쿼리가 약간만 변경되어도 실행 계획을 재사용하지 않는다. 예를 들어
```sql
-- 구문 확인, 쿼리 컴파일, 실행 계획 생성
select name from Employees where id=1;

-- 실행 계획 재사용
select name from Employees where id=1;

-- 실행 계획 재사용x
select name from Employees where id=2; -- 또는
select name from Employees  where id=1; -- where 앞에 한 칸 공백
```
위와 같이 약간만 변경되어도 실행 계획을 재사용하지 않고 새로운 실행 계획을 수립하게 된다.

### 2. 네트워크 트래픽 감소
어떤 SQL 쿼리가 3000줄의 바디를 가지고 있다고 할 때 이러한 덩어리를 매 쿼리를 수행할 때 보내는 것보다 저장 프로시저로 만들어
호출 시에 저장 프로시저 이름과 매개 변수만 보내면 네트워크 소요 값을 줄일 수 있어서 좋다.

### 3. 코드 재사용 및 유지보수 이점
저장 프로시저는 SQL Server에 저장된다. 그로 인한 이점으로 해당 SQL Server를 여러 클라이언트에서 같은 쿼리를 수행한다고 할 때,
쿼리를 수정할 필요가 있는 경우, 저장 프로시저를 사용하지 않으면 각 클라이언트에서 각각 수정을 해줘야 하지만 저장 프로시저를 
사용하는 경우 SQL Server에서만 수정해주면 쉽게 사용할 수 있다.
이 장점 부분은 인라인 SQL과 저장 프로시저의 중요한 차이점이다.

### 4. 보안 향상
두 가지 관점이 있다.
#### 4-1 액세스 권한
예를 들어, 직원 테이블이 있고 이 테이블에 직원의 이름과 연봉 등의 정보들이 저장되어 있는 상황에서 어느 한 직원에게
이 테이블에 대한 접근 권한을 준다면 해당 직원이 다른 직원들의 연봉 정보까지 보게 되는 상황이 발생한다. 이 경우 해당 직원에게
테이블에 대한 접근 권한을 주는 것이 아니라 원하는 정보를 조회할 수 있는 저장 프로시저를 만들고 해당 저장 프로시저에 대한 권한을
할당하는 방식으로 액세스 권한이 있는 적합한 사용자에게만 정보를 제공할 수 있다.

#### 4-2 SQL Injection
입력한 상품을 검색하는 기능이 필요하고 아래와 같이 구현되어 있다고 해보자.
```c++
//
void GetProductByName_BtnOnClicked() {
  SqlConenction conn = new SqlConnection();

  // SQL Injection 유발 코드
  SqlCommand cmd = new SqlCommand("select * from products where name='" + textbox1.text + "'", conn);
}
```
정상적인 사용자가 Book을 입력하면 아래와 같은 쿼리가 수행된다.
```sql
select * from Products where name='Book';
```

악의 적인 사용자가 pens'; delete from Products; -- 을 입력하면 아래와 같은 쿼리가 수행된다.
```sql
select * from Products where name='   pens'; delete from Products; --'
```
이 코드가 어떻게 수행되는가하면 아래와 같이 수행된다.<br/>
select * from Products where name='pens';<br/>
delete from Products;<br/>
-- (뒷 라인 주석 처리)<br/>
Products 테이블의 레코드를 모두 삭제하게 된다.<br/>

# 저장 프로시저 실습
```sql
-- 예제 테이블
create table employees(
id int primary key, identity(1, 1),
gender nvarchar(20),
departmentId int,
entryDate datetime
);
```
### 저장 프로시저 생성 및 호출
```sql
-- 저장 프로시저 생성
create procedure uspGetEmployees
as
begin
select * from Employees
end

-- 저장 프로시저 호출
exec spGetEmployees;
```
### 저장 프로시저 생성 및 호출 (입력 매개변수가 있는 경우)
```sql
-- 저장 프로시저 생성 (입력 매개변수)
create procedure uspGetEmployeesByGenderAndDepartmentId
@Gender nvarchar(20),
@DepartmentId int
as
begin
select * from Employees where
gender=@Gender and DepartmentId=@DepartmentId;
end

-- 저장 프로시저 호출 (입력 매개변수)
-- 매개 변수를 상수로 전달
exec uspGetEmployeesByGenderAndDepartmentId 'Male', 1;

-- 매개 변수를 변수로 전달
declare @Gender nvarchar(20), @DepartmentId int
set @Gender='Male';
set @DepartmentId=1;
exec uspGetEmployeesByGenderAndDepartmentId @Gender, @DepartmentId;

-- 함수를 매개변수로 사용할 수 없기에
exec uspGetEmployeesByEntryDate GetDate();
-- 함수의 값을 변수에 담고 변수로 넘겨줘야 한다.
declare @entryDateTmp datetime;
set @entryDateTmp=GetDate();
exec uspGetEmployeesByEntryDate @entryDateTmp;
```
### 저장 프로시저 생성 및 호출 (출력 매개변수가 있는 경우)
```sql
-- 저장 프로시저 생성 (출력 매개변수)
create procedure uspGetEmployeeCountByGender
@Gender nvarchar(20),
-- output 키워드 지정하여 출력 매개변수 선언
-- output을 지정하지 않을 경우 이 값은 null 처리된다.
@EmployeeCount int output
as
begin
select @EmployeeCount = COUNT(Id) from Employees
where Gender=@Gender
end

-- 저장 프로시저 호출 (출력 매개변수)
declare @TotalCount int
exec uspGetEmployeeCountByGender 'Male', @TotalCount out
if(@TotalCount is null) print '@TotalCount is null'
else print '@TotalCount is not null'

print @TotalCount;
```
### 저장 프로시저 삭제
```sql
drop procedure uspGetEmployees;
```
### 저장 프로시저 변경
```sql
alter procedure uspGetEmployeesByGenderAndDepartmentId
@Gender nvarchar(20),
@DepartmentId int,
WITH Encryption -- 저장 프로시저 텍스트 암호화
as
begin
select * from Employees where gender=@Gender and departmentId=@DepartmentId;
end
```

