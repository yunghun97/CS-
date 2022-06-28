# 🕵️‍♀️ Redis Monitoring

### Redis Info를 통한 정보
- RSS (데이터를 포함해서 실제로 redis가 사용하고 있는 메모리)
- Used Memory
- Connection 수
- 초당 처리 요청 수
  
### System
- CPU
- Disk
- Network rx/tx

### CPU 사용량이 100% 일 경우
- 처리량이 매우 많다면
    - CPU 성능이 좋은 서버로 이전
    - 실제 CPU 성능에 영향을 받음
- O(N) 계열의 특정 명령이 많은 경우
    - Monitor 명령을 통해 특정 패턴을 파악하는 것이 필요
    - Monitor 잘못쓰면 부하로 해당 서버에 더 큰 문제를 일으킬 수도 있음.(짧게 쓰는게 좋음)

## ❗ 결론
- 기본적으로 Redis는 매우 좋은 툴
- 메모리를 빡빡하게 쓸 경우, 관리하기가 어려움
    - 32GB 장비라면 24GB 이상 사용하면 장비 증설을 고려하는 것이 좋음
    - Write Heavy 할 때는 migration도 매우 주의해야함
- Client-output-buffer-limit 설정이 필요

## Redis as Cache
- Cache 일 경우는 문제가 적게 발생
- Redis 가 문제가 있을 때 DB등의 부하가 어느정도 증가하는 지 확인 필요
- Consistent Hashing도 실제 부하를 아주 균등하게 나누지는 않음. Adaptive Consistent Hashing을 이용해 볼 수 도 있음

## Redis as Persistent Store
- 무조건 Primary/Secondary 구조로 구성이 필요함
- 메모리를 절대로 빡빡하게 사용하면 안됨.
    - 정기적인 migration이 필요
    - 가능하면 자동화 툴을 만들어서 이용
    - RDB/AOF가 필요하다면 Secondary에서만 구동
