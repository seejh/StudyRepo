# EF Core
마이크로소프트의 .NET Core 기반의 ORM 프레임워크 <br/>
CRUD, 변경 내용 추적, 스키마 마이그레이션, 스캐폴드 지원 <br/>

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

# 설치
## 패키지
Microsoft.EntityFrameworkCore - EFCore <br/>
Microsoft.EntityFrameworkCore.Tools - EFCore 명령어 <br/>
Microsoft.EntityFramewarkCore.SqlServer - EFCore MSSQL <br/>
# EF Core 간단 이해
## 업데이트 예제
```c#
using var db = new BloggingContext();

// 삽입 (Insert)
db.Add(new Blog { Url = "http://blogs.msdn.com/adonet" });
db.SaveChanges();

// 탐색 (Select)
var blog = db.Blogs
    .OrderBy(b => b.BlogId)
    .First();

// 업데이트 (Update)
blog.Url = "https://devblogs.microsoft.com/dotnet";
blog.Posts.Add(
    new Post
    {
        Title = "Hello World",
        Content = "I wrote an app using EF Core!"
    });
db.SaveChanges();

// 삭제 (Delete)
db.Remove(blog);
db.SaveChanges();
```

## 마이그레이션 예제
```c#
// Models.cs
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;

public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    public string DbPath { get; }

    public BloggingContext()
    {
        var folder = Environment.SpecialFolder.LocalApplicationData;
        var path = Environment.GetFolderPath(folder);
        DbPath = System.IO.Path.Join(path, "blogging.db");
    }

    // The following configures EF to create a Sqlite database file in the
    // special "local" folder for your platform.
    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlite($"Data Source={DbPath}");
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    public List<Post> Posts { get; } = new();
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public int BlogId { get; set; }
    public Blog Blog { get; set; }
}
```

패키지 관리자 콘솔 <br/>

출처 : <br/>
https://learn.microsoft.com/ko-kr/ef/core/get-started/overview/first-app?tabs=visual-studio <br/>

<hr/>


