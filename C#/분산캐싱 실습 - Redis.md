# 예제 설명
ASP.NET Core에서 레디스를 캐시로 사용하는 경우에 대한 예제 <br/>
레디스 연결 방법 및 사용 방법 설명 <br/>
추가로 캐시 용도로서의 기본 사용법 설명 <br/>

# 레디스 서버 프로그램 설치 및 레디스 서버 구동
TODO <br/>
대충 레디스 프로그램 깔고 서버 실행 <br/>

# ASP.NET Core에서 Redis
## 패키지 설치
Microsoft.EntityFrameworkCore.SqlServer - MSSQL 사용 (Redis를 캐시로 사용한다면) <br/>
Microsoft.EntityFrameworkCore.Tools - EF Core 사용 <br/>
Microsoft.Extensions.Caching.StackExchangeRedis - 레디스 사용 <br/>

## 데이터베이스 모델 및 DbContext 정의
루트 디렉토리에 Models 폴더를 생성 후 Product.cs 파일 생성 및 코드 기입 <br/>
루트 디렉토리에 Data라는 폴더를 만든 후 ApplicationDbContext.cs 파일 생성 및 코드 기입 <br/>
여러 예제를 봐왔지만 폴더명칭과 나누는 것은 조금 더 고려할 필요가 있어보임 <br/><br/>

Root/Models/Product.cs <br/>
```c#
namespace RedisCachingDemo.Models
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Category { get; set; }
        public int Price { get; set; }
        public int Quantity { get; set; }
    }
}
```

Root/Data/ApplicationDbContext.cs <br/>
```c#
using Microsoft.EntityFrameworkCore;
using RedisCachingDemo.Models;

namespace RedisCachingDemo.Data
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
        {}

        // Override this method to configure the model and seed initial data
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // 테스트를 위해 DbContext가 생성될 때 샘플 생성 및 삽입
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
                }
            };
            modelBuilder.Entity<Product>().HasData(initialProducts);
        }

        public DbSet<Product> Products { get; set; }
    }
}
```

## 레디스 연결
appsettings.json 파일 수정 <br/>
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=LAPTOP-6P5NK25R\\SQLSERVER2022DEV;
    Database=ProductsDB;Trusted_Connection=True;TrustServerCertificate=True;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "RedisCacheOptions": {
    "Configuration": "localhost:6379",
    "InstanceName": "RedisCachingDemo"
  }
}
```

program.cs 파일 수정 <br/>
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
            builder.Services.AddControllers()
            .AddJsonOptions(options =>
            {
                // This will use the property names as defined in the C# model
                options.JsonSerializerOptions.PropertyNamingPolicy = null;
            });
            builder.Services.AddEndpointsApiExplorer();
            builder.Services.AddSwaggerGen();

            // appsettings.json에서 구성 정보를 가져와서 MSSQL과 레디스 연결
            // MSSQL 추가 (DbContext)
            builder.Services.AddDbContext<ApplicationDbContext>(options =>
                options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
            // 레디스 분산 캐시 추가
            // 컨테이너에 서비스 추가
            builder.Services.AddStackExchangeRedisCache(options =>
            {
                options.Configuration = builder.Configuration["RedisCacheOptions:Configuration"];
                options.InstanceName = builder.Configuration["RedisCacheOptions:InstanceName"];
            });

            // 확실하지 않은 내용
            // 레디스
            builder.Services.AddSingleton<IConnectionMultiplexer>(sp =>
                ConnectionMultiplexer.Connect(builder.Configuration["RedisCacheOptions:Configuration"]));

            var app = builder.Build();
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

## 레디스 사용
Program.cs 클래스에서 레디스 연결 구성을 완료하고 난 뒤, IDistributedCache 서비스 인스턴스를 컨트롤러 또는 서비스에 삽입하여
Redis를 사용할 수 있다. <br/>
### 레디스에 삽입 - SetStringAsync
선택적 캐싱 옵션(예, 만료 정책)에 따라 키와 해당하는 값을 레디스에 저장한다. <br/>
해당 키가 존재하는 경우 새 값으로 업데이트 된다. <br>
예시 코드 <br/>
```c#
/*-----------------------------------------------------------------------------
  DB에서 데이터를 가져온 후 레디스에 캐싱하여 후속 요청에서 더 빠르게 검색할 수 있도록,
  말 그대로 캐싱하는 코드 예제이다.
------------------------------------------------------------------------------*/

// DB의 Products 테이블의 내용을 가져옴
product = await _context.Products.AsNoTracking().ToListAsync();

// 데이터 직렬화 (여기서는 JSON)
string jsonStr = JsonSerializer.Serialize(products);

// 캐싱 옵션 지정 (여기서 슬라이딩 만료 시간 5분)
var cacheOptions = new DistributedCacheEntryOptions
{
  SlidingExpiration = TimeSpan.FromMinutes(5)
};

// 레디스에 저장 (비동기)
await _cache.SetStringAsync("SomeCacheKey", serializedData, cacheOptions);
```

### 레디스에서 탐색 - GetStringAsync
매개 변수로 전달된 키값에 해당하는 값(문자열로 표현)을 검색한다. <br/>
예시 코드 <br/>
```c#
// 레디스에 검색 (비동기)
var cachedData = await _cache.GetStringAsync("SomeCacheKey");

// 결과 조회
if (!string.IsNullOrEmpty(cachedData)) {
    // Cache Hit
    // 역직렬화 후 데이터 사용
}
else {
    // Cache Miss
    // DB에서 데이터를 찾고 추후 요청을 대비해 레디스에 캐싱한다.
}
```

### 레디스에서 제거 - RemoveAsync
지정한 키에 해당하는 항목을 삭제한다. <br/>
예시 코드 <br/>
```c#
// 레디스에서 키에 해당하는 데이터 삭제(비동기) <br/>
await _cache.RemoveAsync("SomeCacheKey");
```

### 레디스 사용 예제
```c#
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Caching.Distributed;
using RedisCachingDemo.Data;
using RedisCachingDemo.Models;
using System.Text.Json;

namespace RedisCachingDemo.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class ProductsController : ControllerBase
    {
        // 생성자에서 ApplicationDbContext와 IDistributedCache 삽입
        private readonly ApplicationDbContext _context;
        private readonly IDistributedCache _cache;
        public ProductsController(ApplicationDbContext context, IDistributedCache cache)
        {
            _context = context;
            _cache = cache;
        }

        // 모든 제품 검색
        // GET: api/products/all
        [HttpGet("all")]
        public async Task<IActionResult> GetAll()
        {
            var cacheKey = "GET_ALL_PRODUCTS";
            List<Product> products;
            try
            {
                // 레디스 캐시에서 검색
                var cachedData = await _cache.GetStringAsync(cacheKey);
                if (!string.IsNullOrEmpty(cachedData))
                {
                    // 캐시 히트, 역직렬화
                    products = JsonSerializer.Deserialize<List<Product>>(cachedData) ?? new List<Product>();
                }
                else
                {
                    // 캐시 미스, DB에서 검색
                    products = await _context.Products.AsNoTracking().ToListAsync();
                    if (products != null)
                    {
                        // 데이터 직렬화 -> 캐시 옵션 지정 -> 레디스에 삽입
                        var serializedData = JsonSerializer.Serialize(products);
                        var cacheOptions = new DistributedCacheEntryOptions()
                            .SetSlidingExpiration(TimeSpan.FromMinutes(5));
                        await _cache.SetStringAsync(cacheKey, serializedData, cacheOptions);
                    }
                }

                // 제품 목록을 담아서 200 OK 응답
                return Ok(products);
            }
            catch (Exception ex)
            {
                // 500 에러 응답
                return StatusCode(500, new {
                    message = "An error occurred while retrieving products.", details = ex.Message
                });
            }
        }

        // 제품 카테고리 검색
        // GET: api/products/Category?Category=Fruits
        [HttpGet("Category")]
        public async Task<IActionResult> GetProductByCategory(string Category)
        {
            // 
            var cacheKey = $"PRODUCTS_{Category}";
            List<Product> products;
            try {
                // 레디스 캐시에서 검색
                var cachedData = await _cache.GetStringAsync(cacheKey);
                if (!string.IsNullOrEmpty(cachedData))
                {
                    // 캐시 히트, 역직렬화
                    products = JsonSerializer.Deserialize<List<Product>>(cachedData) ?? new List<Product>();
                }
                else
                {
                    // 캐시 미스, DB에서 검색
                    products = await _context.Products
                        .Where(prd => prd.Category.ToLower() == Category.ToLower())
                        .AsNoTracking()
                        .ToListAsync();
                    if (products != null)
                    {
                        // 레디스 캐시에 저장
                        var serializedData = JsonSerializer.Serialize(products);
                        var cacheOptions = new DistributedCacheEntryOptions()
                            .SetAbsoluteExpiration(TimeSpan.FromMinutes(5));
                        await _cache.SetStringAsync(cacheKey, serializedData, cacheOptions);
                    }
                }
                // 제품 포함, 200 응답 리턴
                return Ok(products);
            } catch (Exception ex) {}
        }

        // 제품 조회
        // GET: api/products/{id}
        [HttpGet("{id}")]
        public async Task<IActionResult> GetProduct(int id)
        {
            // 
            var cacheKey = $"Product_{id}";
            Product? product;
            try {
                // 레디스 캐시에서 검색
                var cachedData = await _cache.GetStringAsync(cacheKey);
                if (!string.IsNullOrEmpty(cachedData))
                {
                    // 캐시 히트, 역직렬화
                    product = JsonSerializer.Deserialize<Product>(cachedData) ?? new Product();
                }
                else
                {
                    // 캐시 미스, DB에서 검색
                    product = await _context.Products.FindAsync(id);
                    if (product == null)
                        return NotFound($"Product with ID {id} not found.");

                    // 레디스 캐시에 저장
                    var serializedData = JsonSerializer.Serialize(product);
                    await _cache.SetStringAsync(cacheKey, serializedData, new DistributedCacheEntryOptions
                    {
                        SlidingExpiration = TimeSpan.FromMinutes(5)
                    });
                }

                // 제품과 200 응답
                return Ok(product);
            } catch (Exception ex) {}
        }

        // 제품 추가 
        // PUT: api/products/{id}
        [HttpPut("{id}")]
        public async Task<IActionResult> UpdateProduct(int id, [FromBody] Product updatedProduct)
        {
            if (id != updatedProduct.Id) { return BadRequest("Product ID mismatch."); }

            try {
                // DB에서 해당 제품 검색
                var existingProduct = await _context.Products.FindAsync(id);
                if (existingProduct == null)
                    return NotFound($"Product with ID {id} not found.");

                // DB 데이터 변경
                _context.Entry(existingProduct).CurrentValues.SetValues(updatedProduct);
                await _context.SaveChangesAsync();

                // 캐시에 삽입
                var cacheKey = $"Product_{id}";
                var serializedData = JsonSerializer.Serialize(updatedProduct);
                await _cache.SetStringAsync(cacheKey, serializedData, new DistributedCacheEntryOptions
                {
                    SlidingExpiration = TimeSpan.FromMinutes(5)
                });

                // OK 응답
                return Ok();
            }
            catch (Exception ex) {}
        }

        // 특정 제품 삭제
        // DELETE: api/products/{id}
        [HttpDelete("{id}")]
        public async Task<IActionResult> DeleteProduct(int id)
        {
            try {
                // DB에서 제품 탐색
                var product = await _context.Products.FindAsync(id);
                if (product == null)
                    return NotFound($"Product with ID {id} not found.");

                // DB에서 삭제
                _context.Products.Remove(product);
                await _context.SaveChangesAsync();

                // 캐시에서도 제거
                var cacheKey = $"Product_{id}";
                await _cache.RemoveAsync(cacheKey);

                // OK 응답
                return Ok();
            } catch (Exception ex) {}
        }
    }
}
```

### 고급 기법
```c#
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Caching.Distributed;
using StackExchange.Redis;
namespace RedisCachingDemo.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class RedisCacheController : ControllerBase
    {
        // Dependency for interacting with Redis via IDistributedCache
        private readonly IDistributedCache _distributedCache;
        // Dependency for interacting directly with Redis Server 
        private readonly IConnectionMultiplexer _redisConnection;
        // Dependency for interacting with the configuration files like appsettings.json file
        private readonly IConfiguration _configuration;
        // Inject IDistributedCache, IConnectionMultiplexer, and IConfiguration
        public RedisCacheController(IDistributedCache distributedCache, IConnectionMultiplexer redisConnection, IConfiguration configuration)
        {
            _distributedCache = distributedCache;
            _redisConnection = redisConnection;
            _configuration = configuration;
        }

        
        // GET: api/RedisCache/all
        // Retrieve all keys and their values from the Redis cache
        [HttpGet("all")]
        public async Task<IActionResult> GetAllCachedKeysAndValues()
        {
            try
            {
                // Get the first available Redis server from the connection multiplexer
                var server = _redisConnection.GetServer(_redisConnection.GetEndPoints().First());
                // Retrieve all keys from Redis
                var keys = server.Keys().ToArray();
                // Read the instance name prefix from configuration
                string instanceName = _configuration["RedisCacheOptions:InstanceName"] ?? string.Empty;
                // Initialize a list to store key-value pairs from the cache
                var cacheEntries = new List<KeyValuePair<string, string>>();
                // Loop through each key, remove the instance prefix and fetch the cached value
                foreach (var key in keys)
                {
                    // Remove the instance name prefix if it exists
                    var keyWithoutPrefix = key.ToString().Replace($"{instanceName}", "");
                    // Get the value associated with the key from the distributed cache
                    var value = await _distributedCache.GetStringAsync(keyWithoutPrefix);
                    // Add the key-value pair to the list; if value is null, set it to "null"
                    cacheEntries.Add(new KeyValuePair<string, string>(keyWithoutPrefix, value ?? "null"));
                }
                // Return the list of key-value pairs as a JSON response
                return Ok(cacheEntries);
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "Failed to retrieve cache entries.", error = ex.Message });
            }
        }
        // GET: api/RedisCache/{key}
        // Retrieve a specific cache entry by key
        [HttpGet("{key}")]
        public async Task<IActionResult> GetCacheEntryByKey(string key)
        {
            try
            {
                // Attempt to retrieve the value associated with the provided key asynchronously
                var value = await _distributedCache.GetStringAsync(key);
                // If the key is not found in the cache, return a 404 Not Found response
                if (value == null)
                {
                    return NotFound(new { message = "Cache entry not found." });
                }
                // Return the key-value pair as a JSON response
                return Ok(new { Key = key, Value = value });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "Failed to retrieve cache entry.", error = ex.Message });
            }
        }
        // DELETE: api/RedisCache/all
        // Clear all cache entries
        [HttpDelete("all")]
        public IActionResult ClearAllCacheEntries()
        {
            try
            {
                // Get the first Redis server instance from the connection multiplexer
                var server = _redisConnection.GetServer(_redisConnection.GetEndPoints().First());
                // Iterate over all keys in the Redis database
                foreach (var key in server.Keys())
                {
                    // Get the InstanceName from the configuration (as specified in appsettings.json)
                    string instanceName = _configuration["RedisCacheOptions:InstanceName"] ?? string.Empty;
                    // Remove the instance name prefix if it exists
                    var keyWithoutPrefix = key.ToString().Replace($"{instanceName}", "");
                    // Remove each key-value pair from the distributed cache
                    _distributedCache.Remove(keyWithoutPrefix);
                }
                // Return a success message
                return Ok(new { message = "All cache entries cleared." });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "Failed to clear cache entries.", error = ex.Message });
            }
        }
        // DELETE: api/RedisCache/{key}
        // Clear a specific cache entry by key
        [HttpDelete("{key}")]
        public async Task<IActionResult> ClearCacheEntryByKey(string key)
        {
            try
            {
                // Remove the cache entry associated with the provided key asynchronously
                await _distributedCache.RemoveAsync(key);
                // Return a success message indicating the specific cache entry was cleared
                return Ok(new { message = $"Cache entry '{key}' cleared." });
            }
            catch (Exception ex)
            {
                return StatusCode(500, new { message = "Failed to clear cache entry.", error = ex.Message });
            }
        }
    }
}
```

출처 : <br/>
https://dotnettutorials.net/lesson/how-to-implement-redis-cache-in-asp-net-core-web-api/#google_vignette <br/>
<hr/><br/><br/>

# RedisExchange 설명
https://stackexchange.github.io/StackExchange.Redis/Basics.html <br/>

# 레디스 연결하는 방법 분석
https://m.blog.naver.com/bluekms21/222640006011 <br/>
<hr/>
