# EF Core
마이크로소프트의 .NET Core 기반의 ORM 프레임워크 <br/>
CRUD, 변경 내용 추적, 스키마 마이그레이션와 스캐폴드, 버전 관리 <br/>

## 용어 및 추가 설명
### ORM(Object Relational Mapper) 
프로그래밍 언어와 DB 간의 데이터 매핑을 자동으로 처리하는 프레임워크나 라이브러리를 의미 <br/>
위의 내용을 쉽게 설명하자면 아래와 같다. <br/>
1. 프로그래밍 언어에서 DB에 접근(CRUD)하기 위해 SQL 쿼리를 사용하는 대신, <br/>
    매핑된 개체(EFCore에서는 DbContext)를 사용하여 데이터를 가져오거나 업데이트할 수 있다 <br/>
2. 코드 퍼스트, DB 퍼스트의 방법으로 하나의 기준점을 잡고 반대편에 스키마나 프로시저를 자동으로 생성할 수 있다. <br/>
### 코드 퍼스트(Code First) 
프로그래밍 영역에서 모델 클래스를 만들고 이를 바탕으로 DB 개체인 테이블을 만들고 사용하는 방법<br/>
### DB 퍼스트 (DB First) 
SQL 문으로 테이블과 저장 프로시저 등 DB를 먼저 구성하고 프로그래밍 영역(어플리케이션)에 만들고 사용하는 방법 <br/>

### 추가 내용 출처
https://www.dotnetkorea.com/docs/efcore/code-first-vs-database-first/ <br/>

# 실습
C# 콘솔 앱에서 EF Core 사용 예제 </br>
.NET Core 콘솔 앱 프로젝트 생성, 프로젝트 명은 ProductApp <br/>

## 실습 순서
1. Nuget으로 패키지 설치
2. DbContext 세팅
3. 스키마 마이그레이션
4. 기본 CRUD

## 1 패키지 설치
Microsoft.EntityFrameworkCore - EFCore <br/>
Microsoft.EntityFrameworkCore.Tools - EFCore 명령어 <br/>
Microsoft.EntityFramewarkCore.SqlServer - EFCore MSSQL <br/>

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
        // 생성 시 DbContext를 인자로 받으며 멤버로 가지고 있음 
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


실습 출처 : <br/>
https://dev.to/moh_moh701/efcore-tutorial-p1-getting-started-with-ef-core-48g0 <br/>
<hr/><br/><br/>

# 실습 2편
속성(Attribute, Data Annotation), Fluent API를 사용한 엔티티 구성 <br/>

## 속성(Attribute, Data Annotation)을 사용한 엔티티 구성
엔티티 클래스에서 정의되며 쉽고 간단하나 기본 유효성 검사 및 관계로 제한 된다 <br/>

```c#
public class Product
{
    public int Id {get;set;}

    [Required] // Name이 꼭 있어야 함
    [MaxLength(100)] // 문자열 길이 최대 제한
    public string Name {get;set;}

    [Range(0, 10000)] Price의 제한 범위 설정
    public decimal Price {get;set;}

    [ForeignKey("CategoryId")] // Category 테이블에 대한 외래키를 정의
    public Category Category {get;set;}
}
```

## Fluent API를 사용한 엔티티 구성
DbContext에 정의되며 속성(Attribute)이 제공하는 것보다 더 많은 기능을 제공 <br/>
복합키 및 자세한 관계와 같은 고급 구성 지원 <br/>

```c#
public class AppDbContext : DbContext
{
    public DbSet<Product> Products {get;set;}
    public DbSet<Category> Categories {get;set;}
    public Dbset<Inventory> Inventories {get;set;}
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    { optionsBuilder.UseSqlServer("ConenctionString"); }

    protected override void OnModelCreate(ModelBuilder modelBuilder)
    {
        // Category 엔티티의 Fluent API 구성
        modelBuilder.Entity<Category>(entity => {
            // 기본키 설정
            entity.HasKey(c => c.Id);

            // 
            entity.Property(c => c.Name)
                    .IsRequired()
                    .HasMaxLength(50); // Name의 길이를 50 character로 제한

            // Product 테이블과의 관계 구성
            entity.HasMany(c => c.Products)
                    .WithOne(p => p.Category)
                    .HasForeignKey(p => p.CategoryId);
        });
    }
}
```
엔티티를 구성한 후 DB에 마이그레이션 (이미 DB가 구현되어 있으면 패스)<br/>




