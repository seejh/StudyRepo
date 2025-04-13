# EF Core
.NET Core에서 개발자가 DB와 상호 작용할 수 있도록 하는 ORM(개체 관계형 매핑) 프레임워크이다.

EF Core (Entity Framework Core)는 EF Core model과 database schema 사이에 sync를 유지하기 위해 2가지 방법을 제공한다. <br/>
둘 중 우선 되는 것을 정한 뒤 다른 하나를 그 기준에 맞추는 것 <br/>

## EF Core model를 기준 (Migrations) 
데이터베이스의 스키마를 EF Core의 모델과 동기화된 상태로 유지하는 방법 <br/>

#### EF Core 패키지 설치
Microsoft.EntityFrameworkCore.Tools // Migration 명령 입력을 위해 추가 <br/>
Microsoft.EntityFrameworkCore.SqlServer // 데이터베이스 공급자 추가, Sql Server(MSSQL) 사용시 <br/>
Microsoft.EntityFrameworkCore <br/>

#### DbContext 만들기
데이터베이스의 테이블을 나타내는 c# 클래스를 정의하여 
```c#
public class ProductContext : DbContext
{
  protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
  {
    // DB 연결
    optionsBuilder.UseSqlServer("YourConnectionString");
  }
}
```
참고로 DB 연결은 Program.cs에서도 구성할 수 있다.
services.AddDbContext<AppDbContext>(options => options.UseSqlServer("YourConnectionString"));

#### 엔티티 모델 정의
DB 의 테이블을 나타내는 클래스 생성 <br/>
```c#
public class Product
{
  public int ProductId {get;set;}
  public string Name {get;set;}
  public int CategoryId {get;set;}
  public virtual Category Category {get;set;}
}
```
#### 엔티티 구성 정의




## Database Schema가 기준 (Reverse Engineering)



