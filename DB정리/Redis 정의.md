# 😊 Redis 정리

- In-Memory Data Structure
    - 로컬도 가능하지만 일반적으로 프로세스로 존재
- Open Source(BSD 3 License) : 코드를 고치고 숨겨도 상관 없음
    - Redis 모듈은 수정하면 코드 공개해야 함
- 지원하는 데이터 자료구조
    - Strings, set, sorted-set, hashes, list
    - Hyperloglog, bitmap, geospatial index
    - Stream
- Only 1 Committer

- 100ms대의 속도를 1ms미만으로 줄일 수 있다.
- 백엔드 서비스 부하 줄일 수 있다.

### 인메모리
- 데이터를 하드 디스크가 아닌 메인 메모리(RAM)에 모두 올려서 서비스를 수행하는 것
  
### 랭킹 서버
- 가장 간단한 방법
    - DB에 유저의 Score를 저장하고 Score를 저장하고 Score로 Order by로 정렬 후 읽어오기
    - 개수가 많아지면 속도에 문제가 발생할 수 있음.
        - 결국 디스크를 사용하므로
- In-Memory 기준으로 랭킹 서버의 구현이 필요함
    - Redis Sorted Set 또는 Replication 사용
    - 랭킹에 저장해야할 id가 1개당 100byte라 했을 때
        - 10명 1KB
        - 10,000명 1MB
        - 10,000,000명 1GB
        - 10,000,000,000명 1TB
  
### Redis 사용 처
- Remote Data Store
    - A서버, B서버, C서버에서 데이터를 공유하고 싶을때
- 한대에서만 필요하다면 전역 변수 쓰면 되지않나요?
    - Redis 자체가 Atomic을 보장해준다.(주 명령어는 싱글 스레드에서 동작)
- 주로 많이 쓰는 곳들
    - 인증 토큰 등을 저장(Strings 또는 hash)
    - Ranking 보드로 사용(Sorted Set)
    - 유저 API Limit
    - 잡 큐(Salary)

 /*
    Lettuce: Multi-Thread 에서 Thread-Safe한 Redis 클라이언트로 netty에 의해 관리된다.
        Sentinel, Cluster, Redis data model 같은 고급 기능들을 지원하며
        비동기 방식으로 요청하기에 TPS/CPU/Connection 개수와 응답속도 등 전 분야에서 Jedis 보다 뛰어나다.
        스프링 부트의 기본 의존성은 현재 Lettuce로 되어있다.

    Jedis  : Multi-Thread 에서 Thread-unsafe 하며 Connection pool을 이용해 멀티쓰레드 환경을 구성한다.
        Jedis 인스턴스와 연결할 때마다 Connection pool을 불러오고 스레드 갯수가
        늘어난다면 시간이 상당히 소요될 수 있다.
*/


참고자료
---
###### https://www.youtube.com/watch?v=mPB2CZiAkKM
###### https://www.youtube.com/watch?v=Tqaqdfxi-J4
###### http://redisgate.kr/redis/configuration/redis_thread.php