
ASP.NET Core에서 Redis 사용 예제 <br/><br/>

# 설치
## 프로젝트 구조
```
Root
  ㄴModels
    ㄴProduct.cs
  ㄴData
    ㄴApplicationDbContext.cs
```

## 설치할 패키지
Microsoft.EntityFrameworkCore.SqlServer <br/>
Microsoft.EntityFrameworkCore.Tools <br/>
Microsoft.Extensions.Caching.StackExchangeRedis <br/>
.Net Core 앱에서 Redis 캐시를 사용하려면 Microsoft.Extensions.Caching.StackExchangeRedis 패키지가 필요하다. <br/>

## 초기 설정
```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection":
  },
  // 예제와 관련 없는 내용
  "Logging": { .... },
  "AllowedHosts": "",

  // 레디스 서버 정보
  "RedisCacheOptions": {
    "Configuration": "localhost:6379",
    "InstanceName": "RedisCachingDemo"
  } 
}
```

## DB 모델 및 DbContext 정의
루트 디렉토리에 Models 폴더 생성 후, Product.cs 파일 생성, 아래 코드 기입 <br/>
```c#
// Models.cs
namespace RedisCachingDemo.Models
{
  public class Product
  {
    public int Id {get;set;}
    public string Name {get;set;}
    public string Category {get;set;}
    public int Price {get;set;}
    public int Count {get;set;}
  }
}
```

루트 디렉토리에 Data 폴더 생성 후, ApplicationDbContext.cs 파일 생성, 아래 코드 기입 <br/>
```c#
// ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;
using RedisCachingDemo.Models;

namespace RedisCachingDemo.Data
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
        {}

        // 이 함수를 오버라이드하여 모델을 구성하고 초기 데이터를 구성한다.
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // 테스트 용으로 초기 데이터를 넣는다. 각 Product 데이터는 고유한 ID가 있어야 한다.
            var initialProducts = new List<Product>
            {
                new Product
                {
                    Id = 1,
                    Name = "Apple iPhone 14",
                    Category = "Electronics",
                    Price = 999,
                    Quantity = 50
                },
                new Product
                {
                    Id = 2,
                    Name = "Samsung Galaxy S22",
                    Category = "Electronics",
                    Price = 899,
                    Quantity = 40
                },
                new Product
                {
                    Id = 3,
                    Name = "Sony WH-1000XM4 Headphones",
                    Category = "Electronics",
                    Price = 349,
                    Quantity = 30
                }
            };

            // EF Core를 통해 데이터를 Products 테이블에 넣는다. 
            modelBuilder.Entity<Product>().HasData(initialProducts);
        }

        // 이 DbSet은 DB의 Products 테이블에 매핑된다.
        public DbSet<Product> Products { get; set; }
    }
}
```

## Program.cs에서 레디스 캐시와 DbContext 구성
```c#
using Microsoft.EntityFrameworkCore;
using RedisCachingDemo.Data;
using StackExchange.Redis;
namespace RedisCachingDemo
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var builder = WebApplication.CreateBuilder(args);
            // Add services to the container.
            builder.Services.AddControllers()
            .AddJsonOptions(options =>
            {
                // This will use the property names as defined in the C# model
                options.JsonSerializerOptions.PropertyNamingPolicy = null;
            });
            builder.Services.AddEndpointsApiExplorer();
            builder.Services.AddSwaggerGen();

            // SQL Server 데이터베이스에서 사용하도록 DbContext를 구성
            builder.Services.AddDbContext<ApplicationDbContext>(options =>
                options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

            // 레디스 분산 캐시 등록
            builder.Services.AddStackExchangeRedisCache(options =>
            {
                // appsettings.json 파일에서 레디스 연결 문자열을 가져와서 지정한다
                options.Configuration = builder.Configuration["RedisCacheOptions:Configuration"];
                // 레디스 캐시 인스턴스의 논리적 이름을 설정하며 appsettings.json 파일에서 가져와서 지정한다.
                options.InstanceName = builder.Configuration["RedisCacheOptions:InstanceName"];
            });

            // 레디스 연결 멀티플렉서를 싱글톤으로 등록한다.
            // 앱이 레디스와 직접적으로 작용할 수 있도록 한다.
            builder.Services.AddSingleton<IConnectionMultiplexer>(sp =>
                // Establish a connection to the Redis server using the configuration from appsettings.json
                ConnectionMultiplexer.Connect(builder.Configuration["RedisCacheOptions:Configuration"]));

            var app = builder.Build();
            // Configure the HTTP request pipeline.
            if (app.Environment.IsDevelopment())
            {
                app.UseSwagger();
                app.UseSwaggerUI();
            }
            app.UseHttpsRedirection();
            app.UseAuthorization();
            app.MapControllers();
            app.Run();
        }
    }
}
```

## DB 마이그레이션 생성 및 적용
패키지 관리자 콘솔을 열고 Add-Migration 명령을 실행, 새 마이그레이션 파일을 생성한다. <br/>
그 후 Update-Database 명령을 실행, 데이터베이스에 마이그레이션을 적용한다. <br/>
위의 명령을 수행하면 

![image](https://github.com/user-attachments/assets/f8104766-1612-4a34-bd5d-c9cedb92b1c2)



