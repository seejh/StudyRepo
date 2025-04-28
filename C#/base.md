

출처 : <br/>
https://m.blog.naver.com/okcharles/222152730413 <br/>
<hr/><br/><br/>

# Using
C#의 using 용도 2가지 <br/>
```c#
// 1. 파일 참조
using Microsoft.EntityFrameworkCore;
using MyProjectFolder

// 2. 메모리 관리
using (SqlConnection connection = new SqlConnection(ConnectionString))
{
  SqlCommand command = new SqlCommand(queryString, conenction);
  command.Connection.Open();
  command.ExecuteNonQuery();
} // 해당 구문에서 벗어날 때 connection 객체 메모리 해제
```
## 메모리 관리 부분 추가 설명
C#은 자동 메모리 관리 기능을 제공 (Garbage Collector) <br/>
다만 모든 메모리를 다 관리하지는 않는다. (파일, DB, 네트워크 쪽 핸들 및 커넥션은 GC가 자동으로 해제하지 않으므로 직접 정리해야 한다.) <br/>
중요 데이터와 메모리를 빠르게 반납해야 할 때, 다른 프로세스와의 교착 상태를 방지하고자 할 때, 메모리 낭비를 방지하고자 할 때 사용 <br/>
## IDisposalble 인터페이스와 Disponse 패턴
```c#
namespace ProjectName
{
  class SomeObject : IDisposable
  {
    public void Dispose() { Console.WriteLine("메모리 해제 완료"); }
  }

  class Program
  {
    static void Main()
    {
      using (var object : new SomeObject())
      {
        Console.WriteLine("In Block");
      };
      Console.WriteLine("Out Block");
    }
  }
}
/*------------------------------------------------------
  출력
  In Block
  메모리 해제 완료
  Out Block
------------------------------------------------------*/
```

출처 : <br/>
https://blog.naver.com/PostView.nhn?blogId=94cogus&logNo=221539195761 <br/>
<hr/><br/><br/>

# .AsNoTracking()
기본적으로 EF Core는 모든 entity의 상태를 추적(tracking)한다. <br/>
이것은 메모리 사용량을 높이고, 쿼리 속도가 느려진다. <br/>
.AsNoTracking을 쓰면 추적을 안함으로서 위의 문제 <br/>


출처 : <br/>
https://bigexecution.tistory.com/237 <br/>
<hr/><br/><br/>
