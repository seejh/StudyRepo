# EF Core
마이크로소프트의 .NET Core 용 ORM 프레임워크 <br/>
SQL 쿼리를 사용하지 않고 DBMS에 종속적인 코드를 최소화할 수 있다. <br/>
CRUD, 변경 내용 추적, 마이그레이션과 스캐폴드, 버전 관리 등의 기능 이점이 있다. <br/>

## ORM(Object Relational Mapper, (프로그래밍의)객체 (DB의)관계 매퍼)
객체 지향 프로그래밍 언어의 객체와 관계형 데이터베이스 간의 데이터를 자동으로 변환하고 연결하는 매퍼(프레임워크나 라이브러리) <br/>
위의 내용을 쉽게 설명하자면 아래와 같다 (EF Core를 예로 설명) <br/>
1. 프로그래밍 언어에서 DB에 접근(CRUD)하기 위해 SQL 쿼리를 사용하는 대신, <br/>
    매핑된 개체(DbContext)를 사용하여 데이터를 가져오거나 업데이트할 수 있다 
2. 코드 퍼스트, DB 퍼스트의 방법으로 프로그래밍 영역에서 정의된 내용을 바탕으로 DB에 만들거나 <br/>
    DB에 정의된 내용을 바탕으로 프로그래밍 영역에 만듬
### 코드 퍼스트(Code First) 
프로그래밍 영역에서 모델 클래스를 만들고 이를 바탕으로 DB 개체인 테이블을 만들고 사용하는 방법<br/>
### DB 퍼스트 (DB First) 
SQL 문으로 테이블과 저장 프로시저 등 DB를 먼저 구성하고 프로그래밍 영역(어플리케이션)에 만들고 사용하는 방법 <br/>

<hr/><br/><br/>

# 실습 - 기본
C# 콘솔 앱에서 EF Core 사용 예제 </br>
.NET Core 콘솔 앱 프로젝트 생성, 프로젝트 명은 ProductApp <br/>

## 실습 순서
1. 패키지 설치
2. 모델 생성
3. 마이그레이션

## 1 Nuget을 사용한 패키지 설치
프로젝트 우클릭 -> Nuget 패키지 관리 -> 찾아보기 -> 아래 패키지 입력 후 설치 <br/>
패키지를 설치할 프로젝트를 선택해줘야 하고, 프로젝트와 패키지 버전을 맞춰줘야 한다 <br/>
Microsoft.EntityFrameworkCore - EFCore <br/>
Microsoft.EntityFrameworkCore.Tools - EFCore 명령어 <br/>
Microsoft.EntityFramewarkCore.SqlServer - EFCore MSSQL <br/>
Microsoft.EntityFrameworkCore.Design - <br/>

## 모델 생성
```c#
// 엔티티
// /Entities/LogHistory.cs
using System.ComponentModel.DataAnnotations;

namespace LogHistory.Entities;
public class LogHistory
{
    [Key]
    public int Seq { get; set; }
    public string Detail { get; set; }
    public DateTime CreateTime { get; set; } = DateTime.Now;

    public LogHistory(string detail)
    {
        Detail = detail;
    }
}
```

```c#
// DB 컨텍스트
// /DbContexts/FirstAppContext.cs
using EFCoreFirstApp.Entities;
using Microsoft.EntityFrameworkCore;
namespace EFCoreFirstApp.DbContexts;

public class FirstAppContext : DbContext
{
    public DbSet<LogHistory> LogHistories => Set<LogHistory>();

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // "database.db" 파일로 SQLite 사용
        optionsBuilder.UseSqlite("Data Source=database.db");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // LogHistory의 키인 `Seq`는 자동 증가로 설정
        modelBuilder.Entity<LogHistory>()
            .Property(x => x.Seq)
            .ValueGeneratedOnAdd();
    }
}
```

## 마이그레이션
EF Core Tools를 통해 마이그레이션할 수 있으며 엔티티 및 DB 컨텍스트의 변화를 감지해, 변화에 대한 <br/>
마이그레이션 코드를 자동 생성한다. 이 정보로 DB에 업데이트해서 최신의 모델을 데이터베이스 스키마로 적용할 수 있다. <br/>
쉽게 설명하면 엔티티와 DB 컨텍스트의 변화를 DB 스키마에 반영시키는 것 <br/>
### 순서
도구 -> Nuget 패키지 관리자 -> 패키지 관리자 콘솔 열기 <br/>
```
// 패키지 관리자 콘솔에서 입력

// 마이그레이션 생성
// add-migration "이름" 형식이며, 여기서는 마이그레이션 이름을 first로 지정
add-migration first

// 생성한 마이그레이션을 DB 스키마에 적용
update-database

// 위 둘의 명령이 정상 실행되면 프로젝트 디렉토리에 database.db 가 생성된 것을 확인할 수 있으며
// DB에서도 스키마가 잘 적용되어 있는 것을 확인할 수 있다.
// 생성된 database.db는 실행경로로 복사해야 동일한 설정 환경에서 테스트가 가능하다.
```

```c#
// Migrations/20220517050233_first.cs
// add-migration으로 생성된 코드
using System;
using Microsoft.EntityFrameworkCore.Migrations;
#nullable disable

namespace EFCoreFirstApp.Migrations
{
    public partial class first : Migration
    {
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.CreateTable(
                name: "LogHistories",
                columns: table => new
                {
                    Seq = table.Column<int>(type: "INTEGER", nullable: false)
                        .Annotation("Sqlite:Autoincrement", true),
                    Detail = table.Column<string>(type: "TEXT", nullable: false),
                    CreateTime = table.Column<DateTime>(type: "TEXT", nullable: false)
                },
                constraints: table =>
                {
                    table.PrimaryKey("PK_LogHistories", x => x.Seq);
                });
        }

        protected override void Down(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.DropTable(
                name: "LogHistories");
        }
    }
}
```





















## 2 DbContext 세팅
DbContext와 엔티티 모델 생성 <br/>
```c#
// /ProductApp/AppDbContext.cs
using Microsoft.EntityFrameworkCore;

namespace ProductApp
{
    public class AppDbContext : DbContext
    {
        public DbSet<Product> Products { get; set; }
        public DbSet<Category> Categories { get; set; }
        public DbSet<Inventory> Inventories { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            // 아래에 본인의 DB 커넥션 스트링을 넣는다.
            optionsBuilder.UseSqlServer(@"Your_Connection_String_Here");
        }
    }
}
```

```c#
// /ProductApp/Product.cs
namespace ProductApp
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
        public int CategoryId { get; set; }
        public Category Category { get; set; }
    }
}
```

```c#
// /ProductApp/Category.cs
namespace ProductApp
{
    public class Category
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public ICollection<Product> Products { get; set; }
    }
}
```

```c#
// /ProductApp/Inventory.cs
namespace ProductApp
{
    public class Inventory
    {
        public int Id { get; set; }
        public int ProductId { get; set; }
        public Product Product { get; set; }
        public int Quantity { get; set; }
    }
}
```

## 3 마이그레이션
어플리케이션 코드를 기반으로 DB 스키마를 설계하고 생성하겠다는 것(마이그레이션)으로 <br/>
DB에 스키마 등 해당 내용들이 이미 존재한다면 수행하지 않고 패스 <br/>

```
/*----------------------------------------------------------
    도구 -> Nuget 패키지 관리자 -> 패키지 관리자 콘솔 -> 기본 프로젝트를 현재 프로젝트로 설정
    Add-Migration "네이밍" 으로 마이그레이션 생성
    Update-Database로 마이그레이션을 DB에 업데이트
------------------------------------------------------------*/
Add-Migration InitialCreate
Update-Database
```

## 4 CRUD
메서드에 모든 logic을 배치하는 대신 제품 관련 작업을 처리하는 서비스 생성 <br/>
이렇게 하면 코드가 더 깔끔하고 확장 가능해진다 <br/>

```c#
// /ProductApp/ProductService.cs
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;
using System.Linq;

namespace ProductApp
{
    public class ProductService
    {
        private readonly AppDbContext _context;
        public ProductService(AppDbContext context)
        {
            _context = context;
        }

        // 생성
        public void CreateProduct(string productName, decimal price, string categoryName)
        {
            // 새 상품 생성 -> 추가 -> DB에 변경 사항 업데이트
            var category = new Category { Name = categoryName };
            var product = new Product { Name = productName, Price = price, Category = category };
            _context.Products.Add(product);
            _context.SaveChanges();
        }
        // 검색
        public List<Product> GetAllProducts()
        {
            // 
            return _context.Products.Include(p => p.Category).ToList();
        }
        // 업데이트
        public void UpdateProductPrice(int productId, decimal newPrice)
        {
            // 명시한 조건으로 탐색 -> 데이터 변경 -> DB에 변경 사항 업데이트
            var product = _context.Products.FirstOrDefault(p => p.Id == productId);
            if (product != null)
            {
                product.Price = newPrice;
                _context.SaveChanges();
            }
        }
        // 삭제
        public void DeleteProduct(int productId)
        {
            // 명시한 조건으로 탐색 -> 제거 -> DB에 변경 사항 업데이트
            var product = _context.Products.FirstOrDefault(p => p.Id == productId);
            if (product != null)
            {
                _context.Products.Remove(product);
                _context.SaveChanges();
            }
        }
    }
}
```

```c#
// /ProductApp/Program.cs
using ProductApp;
using System;

class Program
{
    static void Main(string[] args)
    {
        using (var context = new AppDbContext())
        {
            // 
            var productService = new ProductService(context);

            // CREATE (생성)
            productService.CreateProduct("Laptop", 999.99M, "Electronics");

            // SELECT (검색)
            var products = productService.GetAllProducts();
            foreach (var product in products)
            {
                Console.WriteLine($"Product: {product.Name}, Category: {product.Category.Name}, Price: {product.Price}");
            }

            // UPDATE (업데이트)
            var productIdToUpdate = products[0].Id;
            productService.UpdateProductPrice(productIdToUpdate, 899.99M);

            // DELETE (삭제)
            var productIdToDelete = products[0].Id;
            productService.DeleteProduct(productIdToDelete);
        }
    }
}
```
<hr/><br/><br/>

# 심화1 - 엔티티
## Entity FrameworkCore에서 Entity
![image](https://github.com/user-attachments/assets/a5351dc4-a65f-4b00-9ba9-be5cee655ac2) <br/>
위의 그림에서 DbContext 클래스에 DbSet<TEntity> 형식으로 포함된 클래스(Department)를 엔티티라고 말한다. <br/>
EF Core는 각 엔티티를 DB 테이블에 매핑하고 엔티티의 각 속성은 DB 테이블의 열에 매핑된다. <br/>
위 예제에서 Department 엔티티는 Departments DB 테이블과 매핑된다. <br/>

## 엔티티 구성 (Attribute, Fluent API)
속성(Attribute, Data Annotation) 방식으로 엔티티 구성, Fluent API 방식으로 엔티티 구성<br/>

### 속성 (Attribute, Data Annotation) 방식
해당 엔티티 클래스 내에서 구성, 간단한 설정만 가능 <br/>
```c#
// /ProductApp/Product.cs, Product 엔티티 구성
public class Product
{
    public int Id {get;set;}

    // Name 값은 꼭 있어야 하고 문자열의 길이는 100자로 제한
    [Required]
    [MaxLength(100)]
    public string Name {get;set;}

    // Price의 값의 범위는 0 ~ 10000이어야 한다.
    [Range(0, 10000)]
    public decimal Price {get;set;}

    // Category 테이블의 외래키(CategoryId) 
    [ForeignKey("CategoryId")]
    public Category Category {get;set;}
}
```

### Fluent API 방식
DbContext에 정의되며 속성(Attribute)이 제공하는 것보다 더 많은 기능을 제공 <br/>
복합키 및 자세한 관계와 같은 고급 구성 지원 <br/>
```c#
// /ProductApp/AppDbContext.cs, Category 엔티티 구성
public class AppDbContext : DbContext
{
    public DbSet<Product> Products {get;set;}
    public DbSet<Category> Categories {get;set;}
    public Dbset<Inventory> Inventories {get;set;}
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    { optionsBuilder.UseSqlServer("ConenctionString"); }

    protected override void OnModelCreate(ModelBuilder modelBuilder)
    {
        // Fluent API 방식으로 Category 엔티티 구성
        modelBuilder.Entity<Category>(entity => {
            // 기본키 설정
            entity.HasKey(c => c.Id);

            // Name은 있어야 하며 문자열 길이를 50자로 제한
            entity.Property(c => c.Name)
                    .IsRequired()
                    .HasMaxLength(50);

            // Product 테이블과의 관계 구성
            entity.HasMany(c => c.Products)
                    .WithOne(p => p.Category)
                    .HasForeignKey(p => p.CategoryId);
        });
    }
}
```
속성, Fluent API 방식으로 엔티티를 구성한 후 DB에 마이그레이션한다 (해당 내용이 이미 DB에 설정되어 있으면 안해도 됨) <br/>

## 엔티티 상태 (Entity State)
상태 5가지 - Added, Modified, Deleted, Unchanged, Detached <br/>
![image](https://github.com/user-attachments/assets/8271284e-6d5d-4984-8522-3e20947be222) <br/>

Unchanged <br/>
DB에는 있고, 수정사항이 없었음
메모리 상의 데이터와 DB 상의 데이터가 동일한 상태

Deleted <br/>


<hr/><br/><br/>

<hr/>
출처 : <br/>
https://www.dotnetkorea.com/docs/efcore/code-first-vs-database-first/ <br/>
https://dev.to/moh_moh701/efcore-tutorial-p1-getting-started-with-ef-core-48g0 <br/>
https://forum.dotnetdev.kr/t/ef-core-6-1/3607 - 설명이 가장 좋다 <br/>
