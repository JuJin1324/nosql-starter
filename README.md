# nosql-starter

## Relational database
### 기존 RDB 의 한계점
> 한계점 1  
> RDB 에 발생할 수 있는 부하는 2가지 있다. 첫번째는 write 에 관한 부하이고 두번째는 read 에 관한 부하이다.  
> read 및 write 공통으로 부하가 발생했을 때 일반적인 해결방법은 DB 서버의 scale-up 이 있다.   
> read 부하를 해결하기 위한 방법으로 Read Replica 를 여러대 생성을 통해서 부하 해결이 가능하다.  
> write 의 부하를 해결하기 위한 방법으로 여러 개의 DB 서버를 sharding 으로 묶어서 해결이 가능하다.
> 위의 두 방법 모두 동적으로 서버를 늘렸다 줄이는 방식인 scale-out 에 유연하지 못하다.  
>
> 한계점 2  
> transaction 이 제공하는 ACID 를 보장하기 위해서 DB 서버의 퍼포먼스에 영향을 미침.  

### Amazon Aurora DB 요금
> 2023.01 기준  
> db.t3.medium   
> CPU/메모리: 2코어/4GB  
> 시간당 요금: 0.125 USD  
> 
> 스토리지 요금: 월별 1GB 당 0.12 USD  
> I/O 요금: 1백만 요청당 0.24 USD  

---

## NoSQL
### 등장 배경
> 대부분이 RDB 가 scale out 이 힘들고 RDB 의 부하 처리 이상의 데이터 트래픽이 발생했을 때 NoSQL 을 등장시켜서
> 문제를 해결하였음.  

### NoSQL 특징
> 1.flexible schema: 스키마 관리를 DB 가 아닌 application 에서 한다.    
> 
> 2.중복 허용(join 회피): 테이블 내의 컬럼이 정규화를 하지 않기 때문에 연관 테이블의 키가 아닌 데이터가 중복되어 들어가고 
> 그로 인하여 해당 연관 테이블의 값이 업데이트된 경우 같이 업데이트 해주어야 한다. 
> 혹은 주문에서의 제품 가격을 생각해보면 제품 A 의 주문 당시 가격이 100원인데 이후 200원이 되었다고 주문 안에서 
> 제품 가격이 변하지는 않는다. 이러한 경우처럼 연관 테이블의 값이 업데이트 되어도 같이 업데이트를 반영할 필요가 없는 경우에 유용할 듯 싶다.  
> 
> 3.scale-out: nosql 은 클러스터링을 구성하여 사용해서 scale-out 에 용이하다.  
> 
> 4.high throughput, low latency: 높은 처리량, 낮은 지연이 특징이다. 단 이를 위해서 데이터의 일관성(consistency)은 일정부분 떨어질 수 있다.

### NoSQL 도입 시기
> 스타트업의 경우 엑세스 패턴(조회를 위한 기준 항목: ex.PK, Index 등)이 자주 변경될 수 있기 때문에 처음에는 RDBMS 를 사용한다.  
> RDBMS 를 사용하다가 트래픽이 점점 상승하는 것을 RDBMS 의 scale-up 을 통해서 대응을 한다.   
> scale-up 의 한계에 다다르면 sharding 혹은 NoSQL 중 선택하여 대용량 트래픽을 대응한다.  

### 참조 사이트
> [NoSQL 설명!! RDB와는 어떤 차이가 있는지도 설명!! MongoDB, Redis 매우 간단한 예제 포함!!](https://www.youtube.com/watch?v=sqVByJ5tbNA)

## DynamoDB
### 등장 배경
> 기존 Amazon 에서 RDB 만 사용하던 시절에 RDB 에 감당할 수 없는 부하가 발생하였음.
> 그런데 조사를 해보니 부하의 약 90% 정도가 join 이 많은 조회라기 보다는 key value 에 적합한 형태의 데이터 조회였음.
> 그래서 DynamoDB 서비스를 도입하며 런칭함.

### 키 디자인 패턴
> DynamoDB 의 테이블은 기본적으로 3개의 파티션을 갖고 시작한다.  
> 각 파티션은 10GB 까지 데이터를 저장할 수 있으며 10GB 가 넘어간 경우 repartitioning 을 통해서 파티션을 추가해서 기존 데이터를 반으로 나누게 된다.
> 하지만 이 모든 것은 직접 처리하는 것이 아닌 DynamoDB 내부에서 하는 것임으로 신경쓸 것은 없다. (비용만 신경쓴다.)  
> 또한 각 파티션은 1000 WCU/sec 혹은 3000 RCU/sec 를 제공하며 둘 중 하나가 초과되어도 파티션이 늘어난다.  
> 
> DynamoDB 의 키는 Partition Key 와 Sort Key 로 구성된다.  
> Partition Key: 파티션 키에 따라 해당 아이템(row) 을 어느 파티션에 저장할 지 지정된다. 일반적으로 검색에 사용된다.
> equal(=) 연산을 통해서만 검색할 수 있다.  
> Sort Key: 검색으로 사용된 Partition Key 의 아이템 중에서 한 번 더 필터하는 기능 및 정렬하는 기능으로 사용된다.
> begin with, between 등의 조건으로 검색할 수 있다.  
>
> DynamoDB 도 transaction 을 제공한다. 하지만 WCU 및 RCU 를 일반적인 Read/Write 시 보다 더 사용함으로 비용을 고려해서 최소한으로 사용하거나 
> 사용하지 않는 것이 좋겠다.
 
### GSI(Global Secondary Index)  
> RDB 의 인덱스처럼 PK 가 아닌 다른 애트리뷰트(RDB 의 칼럼) 로 조회가 필요할 경우 해당 애트리뷰트에 GSI 를 걸어서 사용할 수 있다.  
> Key/Value DB 의 경우 테이블을 특정 조회 목적에 맞게 설계되는 것이 일반적일 수 있다고 생각한다.  
> 그렇기 때문에 일반적으로 애트리뷰트가 아닌 Key 로만 조회가 가능하도록 애플리케이션을 설계하는 것을 대원칙으로 새워야하며,
> 조회의 기준이 되는 항목이 Key 가 아닌 다른 애트리뷰트로 변경하는 경우에는 테이블을 새로 생성하여 기존 테이블을 마이그레이션하는 방식을 고려하는 것이 현명해보인다.  
> 기존의 Key 를 통해서도 조회가 필요하면서 특정 애트리뷰트를 통해서도 같이 조회가 필요한 경우에는 GSI 를 생성을 고려해볼 수 있겠다.  
> 
> LSI(Local Secondary Index)  
> 테이블 생성 시에만 추가할 수 있는 인덱스로 사실 상 사용을 권장하진 않는다.  
> 위에서도 말했든 Key/Value DB 의 경우 사용하는 애플리케이션에서 Key 를 통해서만 조회를 해도 문제 없도록 데이터 엑세스 패턴을 고민하여 
> Key 설계를 할 필요가 있다.  

### Best practice  
> DynamoDB 는 저장된 대용량의 데이터를 Key 를 통해서 특정 데이터 하나 또는 몇개의 아이템을 빠르게 조회하는데에 특화되어 있다.   
> 대량의 Range 쿼리, Full Text Search(단어 또는 구문 검색: like 쿼리), 집계 쿼리 용도로는 비효율적이다.  
> 
> DynamoDB 는 정규화된 여러개의 Entity 가 아닌 하나의 큰 테이블을 지향하여 설계해야한다.  
> 또한 RDB 의 정규화 규칙처럼 공식화된 규칙이 있는 것이 아니기 때문에 테이블 디자인은 사람마다 모두 제각각이 될 수 있다.
> DynamoDB 도입을 위해서는 팀에서 설계 -> 리뷰 -> 설계 -> 리뷰 를 반복적으로 하여 모든 팀원이 DynamoDB 테이블 디자인을 공유해야한다.  

### 비정규화
> RDBMS 의 경우 등장 시 CPU 가격 보다 DISK 가격이 더 비쌌기 때문에 정규화를 통해서 최대한 중복을 제거하여 디스크를 적게 쓰고 join 을 통한 
> CPU 연산을 통해서 데이터를 조합하는 것을 중점적으로 설계되었었다.  
> 
> 하지만 현재 디스크의 GB 당 가격이 1달라 아래로 내려가면서 NoSQL 의 경우에는 join 과 같은 CPU 연산을 줄이고 중복을 허용하여 DISK 를 더 사용하는
> 형태로 구성되었다.
>
> RDBMS 의 경우 데이터 엑세스 패턴을 고민하기 보다는 데이터 모델 정규화 노력이 필요하다. 엑세스 패턴은 테이블 join 을 통해서 어떤 엑세스 패턴이든 대응이 가능하다.  
> 하지만 NoSQL 의 경우 테이블 join 이 없이 큰 테이블을 사용하기 때문에 데이터 엑세스 패턴을 미리 고민하여 설계하는 것이 중요하다.  

### 간단한 PK 유일키
> Primary Key(Partition Key + Sort Key) 로 Partition Key 만 사용하는 경우
> 예시) 사용자 프로필, 세션 스토어, 상품 정보

### Partition Key + Sort Key 를 통한 싱글 테이블 디자인
> 정의: 모든 엔티티를 하나의 테이블로 설계하는 방법
> 
> 장점: 적은 운영 부담, 높은 테이블 최대 성능 및 쓰로틀링 경감
> 
> 단점: 높은 러닝 커브 및 예외 케이스(다른 엑세스 패턴의 엔티티 등)
> 
> 예시)
> 
> |Partition Key|Sort Key: Metadata|Attributes|
> |-------------|-------------------|---------|
> | 1             | Details             | title: hello, artist: jujin, totalDownload: 3 |
> |  1             | Did-1234 | downloadTimestamp: 2022-10-12 11:12:13 |
> |   1            | Did-1235 | downloadTimestamp: 2022-10-12 11:13:13 |
> |    1           | Did-1236 | downloadTimestamp: 2022-10-13 11:13:13 |
> | 2             | Details             | title: hello2, artist: jujin2, totalDownload: 8 |
> |  2             | Did-2234 | downloadTimestamp: 2022-10-12 12:22:13 |
> |   2            | Did-2235 | downloadTimestamp: 2022-10-12 13:23:13 |
> |    2           | Did-2236 | downloadTimestamp: 2022-10-13 12:23:13 |
> 
> Partition Key 는 UUID 혹은 SeqNo 처럼 증가되는 값이고 하나의 Partition Key 안에 Sort Key 를 메타데이터를 저장하도록 만들어서 
> 여러 테이블(엔티티)를 하나의 Partition Key 안에 넣는 것이 가능하다.

### 싱글 테이블 디자인의 안티 패턴
> 싱글 테이블 디자인에서 지양해야하는 포인트들은 다음과 같다.  
> 1.PK 를 UserID 로 고정하고 시작하는 습관   
> DynamoDB 사용의 이유는 RDB 로는 감당이 안되는 대용량 트래픽 처리를 위해서 사용을 한다. 그런데 보통 서비스의 경우 일반적인 사용 패턴을 보이는 유저와 
> 적은 그룹의 헤비 유저가 존재하게 된다. PK 를 UserID 로 고정하게 되면 특정 파티션에만 트래픽이 몰려 효율적인 운영이 힘들어지게 되기 때문에
> 헤비 유저에 대한 트래픽 분산을 고민해야한다.  
> 
> 2.엔티티 별로 테이블을 만드려는 습관   
> DynamoDB 는 테이블 간 join 을 지원하지 않기 때문에 정규화를 통한 엔티티 별 테이블을 나누지 않아야하며 엔티티가 늘어나는 것은 곧 관리 포인트가
> 늘어나는 것이기 때문에 최대한 적은 테이블 나아가서는 싱글 테이블만 사용할 수 있도록 지향해야한다.  
> 
> 3.GSI 를 많이 사용하려는 습관   
> DynamoDB 는 DynamoDB 를 사용하는 애플리케이션에서 적절한 엑세스 패턴을 고려해서 테이블을 설계를 진행해야한다. 
> GSI 추가는 비용의 추가를 유발하기 때문에 최대한 사용을 자제해야한다.  

### Amazon DynamoDB 요금
> 2023.01 기준  
> 스토리지 요금: 매월 25GB 까지 무료로 저장, 이후 월별 1GB 당 0.27075 USD  
> 쓰기 요청 유닛 1백만 건당 1.35556 USD  
> 읽기 요청 유닛 1백만 건당 0.271 USD  

## MongoDB
### 특징
> 비정형화된 데이터 저장: 기존의 RDB 의 정형화된 테이블은 각 서비스를 사용하는 유저의 개인화된 데이터를 유연하게 대처할 수 없다.  
> scale up 및 scale out 에 대한 무중단 증가 가능. 클라우드 환경에 적합: 기존의 RDB 는 scale up 위주이며 사용량에 따른 scale out 확장성이 떨어짐.

### 주의점
> mongoDB 의 경우 DynamoDB 와는 다른 종류의 DB 이다.   
> 그렇기 때문에 테이블 설계 및 엑세스 패턴의 설계가 달라질 수 있으며 RDB 를 대체해서 사용할 수 있을 가능성이 있다.  
> 하지만 NoSQL 은 정해진 규격이 없기 때문에 개발자가 적은 스타트업에서는 RDB 를 통해서 구현 후 mongoDB 는 충분한 학습을 통해서
> 학습한 이후에 여러 개발자들과 함께 테이블 및 엑세스 패턴의 설계를 지속한 후에 사용하는 것을 권장한다.  

### 단점
> RDB 는 스키마를 시스템에 저장하기 때문에 디스크에는 컬럼에 들어가는 값만 저장하면 된다. 하지만 mongoDB 의 경우 스키마가 시스템에 없이 디스크에 스키마가 값과 함께
> 저장됨으로 RDB 에 비해서 디스크에 저장하는 데이터가 3배까지도 많이 차이가 날 수 있다. 또한 Disk IO 로 인하여 성능이 취약해 질 가능성이 있다.    
> 
> mongoDB 의 Modeling Pattern 에는 Embedded Data Model 과 Reference Data Model 이 있다.  
> Embedded Data Model 이 Document 의 항목에 Document 를 추가하여 저장하는 방식으로 대부분의 구글링에서 MongoDB 를 사용시 추천하는 방식이며,
> Reference Data Model 은 RDB 에서 각 Document 를 따로 분리하여 ID(키)로 조인하는 방식이다.  
> MongoDB 의 락은 Document(RDB 에서의 Table) Lock 이 최소 lock 단위이기 때문에 Embedded Data Model 을 사용하게 되면 동시성 측면에서 RDB 에 비해서 불리하다.    

### Amazon DocumentDB 요금
> 2023.01 기준  
> db.t3.medium   
> CPU/메모리: 2코어/4GB  
> 시간당 요금: 0.119 USD  
> 
> 스토리지 요금: 월별 1GB 당 0.12 USD  
> I/O 요금: 1백만 요청당 0.24 USD

## Redis
### 주용도
> RDB 데이터 캐싱  
> 웹 애플리케이션에서 DB 로 데이터를 요청하기 전에 Redis 에서 먼저 확인.   
> Redis 에 있는 경우 애플리케이션은 Redis 에 있는 데이터를 반환받아서 요청 처리(DB 에는 접근 안함).  
> Redis 에 없는 경우 애플리케이션이 DB 에 요청하여 데이터를 가져온 후 Redis 에 저장.  
> Redis 는 RAM 을 사용하는 In-Memory DB 이기 때문에 Disk 를 사용하는 RDB 보다 빠른 요청처리가 가능.  

### 사용처
> 1.Remote Data Store  
> A서버, B서버, C서버에서 데이터를 공유하고 싶을 때 사용.  
> 
> 2.쓰레드 세이프한 데이터 저장 용도.  
> 3.인증 토큰 저장  
> 4.유저 API Limit  

### Amazon Elastic Cache 요금
> 2023.01 기준  
> cache.t3.medium  
> CPU/메모리: 2코어/3GB  
> 시간당 요금: 0.099 USD  
> 네트워크 성능: 최대 5기가비트
