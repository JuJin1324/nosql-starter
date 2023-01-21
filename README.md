# nosql-starter

## Relational database
### 기존 RDB 의 한계점
> RDB 에 발생할 수 있는 부하는 2가지 있다. 첫번째는 write 에 관한 부하이고 두번째는 read 에 관한 부하이다.  
> read 및 write 공통으로 부하가 발생했을 때 일반적인 해결방법은 DB 서버의 scale-up 이 있다.   
> read 부하를 해결하기 위한 방법으로 Read Replica 를 여러대 생성을 통해서 부하 해결이 가능하다.  
> write 의 부하를 해결하기 위한 방법으로 여러 개의 DB 서버를 sharding 으로 묶어서 해결이 가능하다.
> 위의 두 방법 모두 동적으로 서버를 늘렸다 줄이는 방식인 scale-out 에 유연하지 못하다.  
>
> transaction 이 제공하는 ACID 를 보장하기 위해서 DB 서버의 퍼포먼스에 영향을 미침.  

---

## NoSQL
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
> 4.high throughput, low latency: 높은 처리량, 낮은 지연이 특징이다. 단 이를 위해서 데이터의 일관성(consistency)은 잉정부분 떨어질 수 있다.

### 참조 사이트
> [NoSQL 설명!! RDB와는 어떤 차이가 있는지도 설명!! MongoDB, Redis 매우 간단한 예제 포함!!](https://www.youtube.com/watch?v=sqVByJ5tbNA)
