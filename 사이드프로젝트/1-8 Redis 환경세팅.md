## Redis를 활용하는 이유
Anonichat에서 Redis는 대기열 관리, 메세지 기록에 활용한다.

### 대기열 관리
추후 확장성을 고려하여 대기열을 Redis에서 관리하기로 결정했다.
현재는 단일 서버에서 구동이 되고 있지만 추후에는 다중 서버로 운영될 가능성을 고려하여
분산환경에서도 효율적인 대기열을 관리하기 위해 Redis를 활용한다.

### 메세지 기록 관리
메세지 기록이 누적된다면 Redis에서 관리하기에는 큰 부담을 가진다.
하지만 Anonichat은 단일성 채팅으로 채팅방을 나가면 해당 채팅방의 채팅 기록은 삭제되는 로직을 가지고 있다.
또한 채팅 기록이 최근 50개로 한정되기 때문에 Redis로 기록을 관리하는게 큰 부담은 아닐것으로 판단되고
캐싱을 통해 빠른 응답으로 사용자 경험을 높여주는게 더 효과적일 것이라고 생각했다.

## Redis 환경설정

### Docker-compose.yml 추가

```yml
  redis:
    image: redis:7.2-alpine
    container_name: redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
      - ./redis/redis.conf:/usr/local/etc/redis/redis.conf
    networks:
      - elk
      - data
    command: redis-server /usr/local/etc/redis/redis.conf

# Spring 컨테이너 설정 추가
      - SPRING_REDIS_HOST=redis
      - SPRING_REDIS_PORT=6379
      - SPRING_REDIS_PASSWORD=비밀번호
      - SPRING_REDIS_TIMEOUT=2000ms
      - SPRING_REDIS_LETTUCE_POOL_MAX_ACTIVE=8
      - SPRING_REDIS_LETTUCE_POOL_MAX_IDLE=8
      - SPRING_REDIS_LETTUCE_POOL_MIN_IDLE=0
```
- redis.conf 파일을 볼륨에 추가하여 redis 컨테이너에서 설정 파일을 참조하여 활용하도록 설정
- Spring 컨테이너에 Redis 관련 설정 추가

### redis.conf 파일 생성
docker-compose.yml이 있는 폴더에 redis 폴더 생성 > redis.conf 파일 생성
```
# 기본 설정
bind 0.0.0.0 //모든 IP 접속 가능(Docker 환경에 적합)
port 6379
timeout 0  

# 비밀번호 설정
requirepass 비밀번호

# AOF 설정 (데이터 영속성)
appendonly yes //AOF(Append only File) 활성화
appendfilename "appendonly.aof" //aof 파일명
appendfsync everysec //1초마다 동기호

# AOF Rewrite 자동 설정
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-rewrite-incremental-fsync yes

# 메모리 관리
maxmemory 256mb //Redis가 사용할 수 있는 최대 메모리
maxmemory-policy allkeys-lru //메모리 부족 시 정책(모든 키 중 가장 오래된 것 삭제)

# 로그 설정
loglevel notice
logfile ""

# RDB 최적화 rdbcompression yes # 압축 활성화 (30-70% 절약) 
rdbchecksum yes # 체크섬으로 무결성 보장 
dbfilename "dump.rdb" # RDB 파일명
stop-writes-on-bgsave-error yes

# 백그라운드 저장 설정(RDB 스냅샷 설정)
save 900 1 //15분 내 1개 변경시 저장
save 300 10 //5분 내 10개 변경시 저장
save 60 10000 //1분 내 10000개 변경시 저장

# 기타 보안 설정
protected-mode yes //보안 모드 활성화
```
### **AOF(Append only File)**
모든 쓰기 명령어를 파일에 기록하는 방식
redis는 메모리에 저장해서 사용하는 방식이기 때문에 영속성을 보장 받을 수 없다.
따라서 서버를 재시작하거나 다운된다면 저장되어 있던 데이터가 모두 사라지게되므로 이를 해결하기 위해 AOF 방식을 사용한다.

실행한 명령어를 기록해뒀다가 재시작시 기록된 명령어를 순서대로 다시 실행해서 데이터를 복구한다.
예시
```
# 사용자가 실행한 명령어들이 그대로 기록됨 
SET user:123 "김철수" 
SET chat:room1 "안녕하세요"

# Redis 재시작 시 
# 1. appendonly.aof 파일을 읽음 
# 2. 위 명령어들을 순서대로 다시 실행

# 결과 
redis> GET user:123 "김철수"  복구됨! 
redis> GET chat:room1 "안녕하세요"  복구됨!
```

**AOF의 단점**
1. 파일 크기가 계속 커짐  
2. 복구 시간이 오래 걸림 (명령어 재실행)  
3. 디스크 I/O 부담

**AOF 데이터 누적 방식**
AOF는 기존 파일에 명령어가 실행될 때마다 추가되는 방식(한 개의 파일에 덮어쓰기하는 형식)
따라서 시간이 지날수록 파일의 크기가 점점 커진다.

예시
```
# 초기 상태 (파일 크기: 0KB)
appendonly.aof → [비어있음]

# 첫 번째 명령어 실행
redis> SET user:1 "김철수"
appendonly.aof → SET user:1 "김철수"  (파일 크기: 50B)

# 두 번째 명령어 실행  
redis> SET user:2 "이영희"
appendonly.aof → SET user:1 "김철수"
                SET user:2 "이영희"  (파일 크기: 100B)

# 세 번째 명령어 실행
redis> SADD online "user:1"  
appendonly.aof → SET user:1 "김철수"
                SET user:2 "이영희"
                SADD online "user:1"  (파일 크기: 150B)

# 계속 이런 식으로 파일 끝에 추가 (Append)
# 파일 크기가 계속 커짐

# 1시간 후
appendonly.aof → [수천 개의 명령어들...]  (파일 크기: 10MB)

# 하루 후  
appendonly.aof → [수만 개의 명령어들...]  (파일 크기: 100MB)

# 일주일 후
appendonly.aof → [수십만 개의 명령어들...]  (파일 크기: 1GB)
```

다른 문제점으로는 같은 키에 대한 중복 명령어들이다.
예시
```
# 문제: 같은 키에 대한 중복 명령어들
SET user:1 "김철수"
SET user:1 "김철수_수정"  
SET user:1 "김철수_최종"
DEL temp_data
SET temp_data "임시"
DEL temp_data  # 결국 삭제된 데이터

# 결과: 파일에는 불필요한 명령어들이 쌓인다.
```

이를 해결하기 위해 AOF Rewrite(압축방식)를 사용한다.

**AOF Rewrite**
AOF Rewrite를 사용하면 각 키의 최종 상태만 새 파일에 기록하여 효율적인 명령어 관리가 가능해진다.
예시
```
# AOF Rewrite 과정

# Before (기존 AOF 파일 - 1GB)
SET user:1 "김철수"
SET user:1 "김철수_수정"  
SET user:1 "김철수_최종"  ← 최종 값만 중요
SET user:2 "이영희"
SADD online "user:1"
SADD online "user:2"
DEL temp_data
SET temp_data "임시"
DEL temp_data  ← 결국 삭제됨
...수천개 명령어...

# Redis AOF Rewrite 실행
redis> BGREWRITEAOF
Background append only file rewriting started

# After (새로운 AOF 파일 - 100MB)
SET user:1 "김철수_최종"  ← 최종 결과만
SET user:2 "이영희"
SADD online "user:1" "user:2"  ← 한번에 처리
# temp_data는 아예 없음 (삭제되었으니까)

# 과정 상세
1. Redis가 현재 메모리 상태를 분석
2. 새로운 임시 AOF 파일 생성 (appendonly.aof.tmp)  
3. 각 키의 최종 상태만 새 파일에 기록
4. 기존 appendonly.aof → appendonly.aof.old 로 백업
5. appendonly.aof.tmp → appendonly.aof 로 교체
6. 백업 파일 삭제
```

### **RDB(Redis Database) 스냅샷 설정**
AOF의 단점을 보완하는 또 다른 영속성 보장 방법
AOF가 명령어를 저장해서 데이터를 보존하는 방법이었다면
특정 시점의 데이터를 통째로 바이너리 형식으로 저장하는 방법

파일 크기가 작고 복구가 빠르다는 장점이 있다.

```
# RDB 스냅샷이란?
# "특정 시점의 메모리 상태를 통째로 파일에 저장"

# 메모리 상태 (예시)
Redis Memory:
├── user:123 → "김철수"
├── user:456 → "이영희"  

# RDB 스냅샷 생성 시점
save 900 1  # 15분 내 1개 변경 시 → 스냅샷

# dump.rdb 파일 생성
# (바이너리 형태로 메모리 전체 상태 저장)

# Redis 재시작 시
# 1. dump.rdb 파일 로드
# 2. 메모리에 모든 데이터 한번에 복원
# 3. 빠른 복구 완료!

# 차이점 비교
AOF: SET user:123 "김철수" ← 명령어 기록
     SET user:456 "이영희"
     ...

RDB: [바이너리 데이터] ← 결과만 저장
     user:123="김철수", user:456="이영희"...
```

**RDB 단점**
1. 데이터 손실 위험 (마지막 스냅샷 이후)  
2. 스냅샷 생성 시 성능 저하  
3. 바이너리라 사람이 읽기 어려움

**RDB 데이터 저장 방식**
AOF와 다르게 RDB는 매번 새로운 파일을 생성해서 저장하는 방식
예시
```
# RDB 스냅샷 생성 과정

# 초기 상태
/data/dump.rdb → [2시간 전 스냅샷]  (10MB)

# 조건 만족 (save 300 10 - 5분간 10개 변경)
redis> SET user:100 "새사용자"  # 10번째 변경
# → 스냅샷 조건 만족! 

# RDB 스냅샷 생성 시작
1. Redis Fork 프로세스 생성 (메모리 복사)
2. 새로운 임시 파일 생성: dump.rdb.tmp
3. 현재 메모리 전체 상태를 dump.rdb.tmp에 저장
4. 압축하여 바이너리 형태로 저장

# 저장 완료 후 원자적 교체
5. dump.rdb.tmp → dump.rdb 로 이름 변경 (원자적 연산)

# 결과
/data/dump.rdb → [방금 생성된 최신 스냅샷]  (12MB)

# 중요한 점: 기존 파일 수정 X, 새 파일 생성 후 교체 O

# 만약 저장 중 실패하면?
dump.rdb.tmp만 삭제되고 기존 dump.rdb는 안전하게 보존

# 스냅샷 생성 주기
14:00 → dump.rdb (10MB)  
14:05 → dump.rdb (12MB) ← 새로 생성됨
14:10 → dump.rdb (11MB) ← 또 새로 생성됨

# 각각 독립적인 완전한 스냅샷!
```
스냅샷이 생성될 때 마다 기존 스냅샷과 교체하는 방식

### Springboot 설정 추가
build.gradle에 내용 추가
```
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

springboot application.properties에 Redis 관련 내용을 추가한다.
```
# Redis  
spring.data.redis.host=${SPRING_REDIS_HOST}  
spring.data.redis.port=${SPRING_REDIS_PORT}  
spring.data.redis.password=${SPRING_REDIS_PASSWORD}  
spring.data.redis.timeout=${SPRING_REDIS_TIMEOUT}  
spring.data.redis.database=0  
  
# Redis Lettuce 연결 풀 설정  
spring.data.redis.lettuce.pool.max-active=${SPRING_REDIS_LETTUCE_POOL_MAX_ACTIVE}  
spring.data.redis.lettuce.pool.max-idle=${SPRING_REDIS_LETTUCE_POOL_MAX_IDLE}  
spring.data.redis.lettuce.pool.min-idle=${SPRING_REDIS_LETTUCE_POOL_MIN_IDLE}  
spring.data.redis.lettuce.pool.max-wait=-1ms  
spring.data.redis.lettuce.shutdown-timeout=100ms  
  
# cache 설정  
spring.cache.type=redis  
spring.cache.redis.time-to-live=86400  
spring.cache.redis.cache-null-values=false
```

