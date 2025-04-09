DataContext 클래스는 Entity Framework를 통해 앱 데이터에 접근하려고 사용 <br/>
Entity Framework의 DbContext 클래스를 상속 받고 유저 공용 속성을 가진다. <br/>

```c#

namespace WebApi.Helpers;

using Microsoft.EntityFrameworkCore;
using WebApi.Entities;

public class DataContext : DbContext
{
    public DbSet<User> Users { get; set; }

    private readonly IConfiguration Configuration;

    public DataContext(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        // 편하게 API를 테스트할 수 있도록 실제 DB 없이 메모리 DB 사용
        options.UseInMemoryDatabase("TestDb");
    }
}

```
