

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

# .AsNoTracking
성능 향상 용도, 메모리에 추적된 복사본을 만들지 않고 DB에서 데이터를 검색하는데 사용되는 방법 <br/>
엔티티 추적을 사용하도록 설정하면 EF Core는 결과 집합의 각 엔티티에 대한 프록시 객체를 만든다. <br/>
이러한 프록시 개체는 엔티티에 대한 변경 내용을 추적하는데 사용되며, SaveChanges()가 호출될 때 DB에 업데이트한다. <br/>
AsNoTracking()을 사용하는 경우 EF Core는 엔티티에 대한 프록시 개체를 만들지 않고 데이터를 검색한다. <br/>

## 사용하는 경우
DB에서 데이터를 읽기만 하고 엔티티를 변경하지 않으려는 경우 <br/>

## 사용하지 않는 경우
엔티티를 변경하고 DB에 다시 저장해야 하는 경우 <br/>

많은 양의 데이터를 검색할 때 성능을 향상시키고 메모리 사용량을 줄일 수 있다. <br/>


출처 : <br/>
https://velog.io/@dotnetdevel/r72o0fwy <br/>
<hr/><br/><br/>
