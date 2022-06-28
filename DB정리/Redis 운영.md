# 🤦‍♂️ Redis 운영
1. **메모리 관리를 잘하자**
2. **O(N) 관련 명령어는 주의하자.**
3. **Replication**
4. **권장 설정 Tip**

## 😀 메모리 관리
- Redis 는 In-Memory Data Store
- Physical Memory 이상을 사용하면 문제가 발생
    - Swap이 있으면 Swap 사용으로 해당 메모리 page 접근 마다 늦어짐.
    - Swap이 없다면? O(N) 죽을 가능 성
- Maxmemory를 설정하더라도 이보다 더 사용할 가능성이 큼.
    - 보통 jemalloc 사용 but jemalloc때문에 Redis는 본인이 사용하는 정확한 메모리 용량을 모름
    - 4095Byte 요청 -> 할당 -> 4097Byte 요청 -> 새로운 페이지 할당 -> 현재 8Kb를 사용하게 됨(4097Byte만 쓰면 되는데)
- RSS 값을 모니터링 해야함.

### 메모리 관리
- 큰 메모리를 사용하는 instance 하나보다는 적은 메모리를 사용하는 instance 여러개가 안전함.
    - fork를 했을 때  Copy-on-Write를 통해 Read만 있을 경우는 상관없지만 Write가 일어나면 메모리 복사를 진행하여 메모리를 더 소모한다. ex) 1.x배 ~ 2배
- Redis는 메모리 파편화가 발생할 수 있음. 4.x대 부터 메모리 파편화를 줄이도록 jemlloc에 힌트를 주는 기능이 들어갔으나, jemalloc 버전에 따라서 다르게 동작할 수 있음
- 다양한 사이즈를 가지는 데이터 보다는 유사한 크기의 데이터를 쓰는 사용하는 경우가 관리에 용이

### 메모리가 부족할 때는
- Cache is Cash
    - 좀 더 메모리 많은 장비로 Migration
    - 메모리가 빡빡하면 Migration 중에 문제가 발생할수도..
- 있는 데이터 줄이기
    - 데이터를 일정 수준에서만 사용하도록 특정 데이터를 줄임,
    - 다만 이미 Swap 사용중이라면 프로세스를 재시작 해야함

### 메모리를 줄이기 위한 설정
- 기본적으로 Collection 들을 다음과 같은 자료구조를 사용
    - Hash -> HashTable을 추갈조 사용
    - Sorted Set -> Skiplist와 HashTable을 이용
    - Set -> HashTable 사용
    - 해당 자료 구조는 메모리를 많이 사용함
- Ziplist 이용

## Ziplist 구조
- In-Memory 특성 상, 적은 개수라면 선형 탐색을 하더라도 빠르다.
- List, hash, sorted set 등을 ziplist로 대체해서 처리를 하는 설정이 존재

## 😮 O(N) 관련 명령어는 주의하자
- Redis 는 Single Threaded.
    -  Redis 는 동시에 1개의 명령을 처리할 수 있다.
    - 단순한 get/set의 경우, 초당 10만 TPS 이상 가능(CPU 속도에 따라)
    - 따라서 1초가 걸리는 명령어를 작성하면 뒤에 다른 작업들은 다 1초가 대기를 해야 한다.

### 대표적인 O(N) 명령어
- KEYS
- FLUSHALL, FLUSHDB
- Delete Collections
- Get All Collections

### Keys 대체 (scan 사용)
- 기존의 keys는 key 값을 가져오는 동안 다른 명령어를 수행하지 못합니다.
- 하지만 scan 명령어는 Keys 처럼 한번에 데이터를 읽는 것이 아니라 긴 명령은 짧은 여려번의 명령으로 바꿀 수 있다.

### Collection의 모든 item을 가져와야 할 때
- Collection의 일부만 가져오거나
    - Sorted Set
- 큰 Collection을 작은 여러개의 Collection으로 나눠서 저장

### Spring security oauth RedisTokenStore 이슈
- Access Token 저장을 List(O(N)) 자료구조를 통해서 이루어짐
    - 검색, 삭제시에 모든 item을 매번 찾아봐야 함
        - 100만개 넘어가면 전체 성능에 영향을 줌
    - 현재는 Set(O(1))을 이용해서 검색, 삭제를 하도록 수정되어있음.

## 🤵 Redis Replication
- Async Replication
    - Replication Lag이 발생할 수 있다.
- Replicaof or 'slaveof' 명령으로 설정가능
    -Replicaof hostname port
- DBMS로 보면 statement replication과 유사

### Replication 설정 과정
- Secondary에 replicaof or slaveof 명령을 전달
- Secondary는 Primary에 sync 명령 전달
- Primary는 현재 메모리 상태를 저장하기 위해
    - Fork
- Fork한 프로세서는 현재 메모리 정보를 disk에 dump
- 해당 정보를 Secondary에 전달
- Fork 이후의 데이터를 Secondary에 계속 전달

### Replication 시 주의할 점
- Replication 과정에서 fork가 발생하므로 메모리 부족이 발생할 수 있다.
- Redis-cli --rdb 명령은 현재 상태의 메모리 스냅샷을 가져오므로 같은 문제를 발생시킴
- AWS나 클라우드의 Redis는 좀 다르게 구현되어서 좀더 해당 부분이 안정적(해당 기능 튜닝)
- 많은 대수의 Redis 서버가 Replica를 두고 있다면
    - 네트웍 이슈나, 사람의 작업으로 동시에 replication이 재시도 되도록 하면 문제가 발생할 수 있음
    - 30GB 네트웍에 100GB를 사용하려 하면 연결이 끊어지고 재시도되는 과정 반복

### redis.conf 권정 설정 Tip
- Maxcient 설정 50000 (Maxclient 값만 네트워크 접속 가능)
- RDB/AOF 설정 off (성능상 유리 안정성 좋음)
- 특정 commands disable
    - keys
    - AWS의 ElasticCache는 이미 하고 있음
- 전체 장애의 90% 이상이 KEYS와 SAVE 설정을 사용해서 발생
