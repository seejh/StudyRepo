db 리플리케이션에 대해서 대충 알아본 내용. 개인적으로 사용할 일이 없을 것 같아 중간에 포기했지만 일단 기록. <br/>
내용이 제대로 있는 것은 아니고 일단 키워드 저장 <br/>

# 리플리케이션
여러 db를 권한에 따라 수직적으로 연결, 마스터는 쓰기 작업만 하고 슬레이브는 읽기 작업만 처리한다. 
마스터와 슬레이브 간 데이터 무결성 검사를 하지 않는 비동기 방식으로 노드들 간의 데이터를 동기화한다.

* 장점<br/>
읽기 성능을 높일 수 있고 지연 시간이 없음
* 단점<br/>
노드들 간의 데이터 동기화가 보장되지 않아, 일관성 있는 데이터를 얻지 못할 수 있다.
마스터가 다운되면 복구 및 대처가 까다롭다.

# 클러스터링
여러 DB를 수평적인 구조로 구축하는 방식. failover 시스템을 구축하기 위해 사용한다.
db들 간 무결성 검사를 하는 동기 방식으로 데이터를 동기화한다. 항상 가용할 수 있게 하는 active-active 방식과
일부 클러스터는 대기 상태로 구성하는 active-standby로 나뉜다.

* 장점<br/>
항상 일관성 있는 데이터를 얻을 수 있고 노드 하나가 다운되더라도 시스템을 장애 없이 운영 가능
* 단점<br/>
노드들 간 데이터 동기화 시간이 필요해 리플리케이션에 비해 쓰기 성능이 떨어진다.
데이터 동기화에 의해 스케일링에 한계가 있다.

# 페일 오버(failover)?
장애 대비 기능. 장애가 발생 시 예비 시스템으로 자동 전환하는 기능이다. 추가적인 자원을 필요로 한다. 이처럼 같은 시스템을 2개 준비하는 것을 이중화라고 한다.

# binlog
서버 내 발생되는 모든 변경 내역이 기록되는 파일
PIT (시점 복구)
안 쓰면 활성화할 필요 없다. IO 부담을 가중시키기 떄문

GTID
데이터베이스로 커밋되는 각 트랜잭션과 함께 생성되고 트랜잭션에 연결되는 고유한 식별자이다. GTID를 사용하면 각 트랜잭션들은 고유한 전역 식별자를 가진다.
완전한 트랜잭션 기반이다. row-based replication의 성능이 더 좋다.

위에서 리플리케이션과 클러스터링이라는 기술을 소개하고 있고 리플리케이션은 성능, 클러스터링은 안정성으로 각각 용도가 다른 느낌이다.

<hr/><br/><br/>

# 게임 서버에서 데이터를 db에 저장하는 방법
## 1. 메모리에서 관리하다가 주기적으로 db에 저장
DB의 속도는 메모리의 속도보다 느릴 수 밖에 없고 메모리의 처리와 DB의 처리를 1대1로 했을 때 실시간 반응성을 요구하는 게임에서는
지연이 생길 수 밖에 없고 이러한 지연은 곧 사용자 경험 저하로 이어진다. 

### 구체적인 구조
1. 인메모리에서 관리<br/>
플레이어 위치, 체력, 버프 상태, 스킬 사용 등은 전부 서버 메모리에서 유지.
C++ 객체로 관리(Pliayer, Inventory, Monster 등)하며 상태 변경은 즉기 객체에 반영한다.(빠름, 응답성 우수)

2. 변경 사항을 모아서 기록 (Dirty Flag or Queue 방식)<br/>
변경이 발생한 객체에 dirty 플래그 설정.
또는 변경 로그를 큐(리스트, 구조체 등)에 저장
```c++
// ex
player->gold += 100;
player->mark_dirty("gold");
```

3. 주기적/이벤트 기반 동기화<br/>
예를 들어 10초마다 한 번 모든 dirty 데이터를 DB에 저장하거나 로그아웃, 맵 이동 등 특정 시점에만 동기화.
백그라운드 스레드 또는 별도 저장 스레드에서 처리한다.
```c++
for (auto& player : players) {
	if (player.is_dirty()) {
		save_to_db(player);
		player.clear_dirty();
	}
}
```
4. 복구/백업 용도로만 DB 사용<br/>
DB는 실시간 로직에는 개입하지 않는다.<br/>
서버 재시작 시 DB에서 복원해 메모리 객체 초기화<br/>

### 주의할 점
1. 데이터 유실 가능성<br/>
서버가 다운되기 전에 저장하지 못한 변경 내용은 유실될 수 있다. 저장 간격의 주기를 줄이거나, **중요 이벤트는 즉시 저장**
2. 멀티 서버 환경에서의 충돌<br/>
플레이어가 여러 서버에 접속 가능한 구조라면 인메모리 동기화에 어려움이 있다. 이러한 경우
분산 락, 중앙 세션 관리, Redis 기반 동기화를 고려해야한다.
4. DB 저장 타이밍 설계 <br/>
	* 주기 저장 : 성능에 좋지만 유실 가능성이 있다.
 	* 이벤트 저장 : 안전하지만 이벤트 설계 복잡, 실무에서는 이 둘을 혼합해서 사용한다.

## 2. db에 바로 바로 저장
유저가 몬스터를 처치 후 아이템을 보상 받는 과정을 위 1번처럼 행했을 때, 서버가 다운되면 유실되는 문제점이 있으므로 1번 방법은 사용할 수 없다.


<hr/><br/><br/>

# 보상 트랜잭션 로그 기반 구조
1. 보상을 지급하기 전에 먼저 DB나 별도 로그 테이블/큐에 보상 지급 예정 내역을 기록
2. 그 후 실제 보상을 게임 서버 메모리와 DB에 적용
3. 성공 시 보상 트랜잭션 상태를 완료로 업데이트, 실패 시 중간에 서버가 죽더라도 로그를 기준으로 지급 여부를 복구 처리하는 구조

## 장점
* 지급 성공 여부가 DB에 확실히 기록된다.
* 서버가 죽어도 보상 재처리 가능하다.
* CS(고객 지원) 문의 시에도 지급 여부 추적이 가능하다.

## C++ MySQL 보상 트랜잭션 처리 흐름
1. 테이블 구조 예시<br/>
```sql
create table reward_transactions (
id bigint auto_increment primary key,
player_id int not null,
item_id int not null,
quantity int default 1,
status enum('PENDING', 'COMPLETED', 'FAILED') default 'PENDING',
created_at TIMESTAMP default CURRENT_TIMESTAMP,
updated_at TIMESTAMP default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP
);
```

2. C++ 코드 흐름 예제 (libmysqlclient 사용 기준)
```c++
bool give_reward(MYSQL* conn, int player_id, int item_id, int quantity) {
	// 1. 트랜잭션 시작
	mysql_autocommit(conn, 0);

	// 2. 보상 트랜잭션 로그 테이블에 INSERT
	const char* insert_sql =
	"INSERT INTO reward_transactions(player_id, item_id, quantity, status) VALUES(?,?,?,'PENDING')";

	// 3. 실제 인벤토리 테이블에 아이템 추가
	const char* reward_sql =
	"UPDATE player_inventory SET quantity = quantity + ? WHERE player_id=? AND item_id=?"

	// 4. 보상 트랜잭션 상태를 COMPLETED로 변경
	const char* update_tx_sql =
	"UPDATE reward_transaction SET status = 'COMPLETED' WHERE id=?";

	// 5. 커밋
	mysql_commit(conn);
	mysql_autocommit(conn, 1);
}
```




