# EF Core
.NET 용 DB 매퍼 <br/>
변경 내용 추적, 업데이트 및 스키마 마이그레이션 등을 지원 <br/>

# 설치
## 패키지

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


