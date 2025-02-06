# Redis Persistence RDB

## 특징

- RDB는 특정 시점의 데이터를 저장하는 방식으로, Redis의 데이터를 디스크에 스냅샷 형태로 저장한다.
- RDB는 데이터 백업 및 복구에 유리하고, 일반적으로 Cold Start 용도로 사용된다.
- AOF 방식보다 디스크 I/O 부하가 적어 성능에 미치는 영향이 적다.
- 저장 주기 시점과 어긋나게 Redis가 다운될 경우 일정 데이터의 유실 가능성이 있다.
- 설정을 통해 특정 조건에만 스냅샷을 저장하도록 조정 가능하다.
- 자주 변경되는 데이터가 많으면 짧은 스냅샷 생성 주기를 권장한다.
- 트랜잭션이 중요한 서비스는 AOF와 함께 사용을 권장한다.
- 대규모 데이터 처리 서비스에서는 AOF와 함께 사용하는 경우가 많다.
- 클러스터 환경에서 RDB 백업이 필요하다면 slave 노드에서 수행하며 master 노드의 성능 저하를 방지한다.

## RDB(Snapshot) 설정

- redis.conf에서 설정한다.
- SAVE 900 1
- SAVE 300 10
- SAVE 60 10000

## 도커 컨테이너 올리기

### Dockerfile

```
FROM redis:latest
COPY ./volume/config/redis.conf /usr/local/etc/redis/redis.conf
CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]
```
```
docker build -t my-redis .
```

### Docker run

```
docker run -d ^
    -p 6379:6379 ^
    -v c:\\workspace\\lab-redis\\volume\\data:/data ^
    --name docker_redis ^
    my-redis:latest

docker exec -it docker_redis /bin/bash
```

## RDB 파일 생성 위치 조회

```
redis-cli CONFIG GET dir
redis-cli CONFIG GET dbfilename
```

![alt text](20250206_161443.png)

## RDB 파일 즉시 생성

```
SET user:1 "Alice"
SET user:2 "Bob"
SAVE
```

![alt text](20250206_161640.png)

![alt text](20250206_161657.png)

![alt text](20250206_163050.png)

## 설정 주기에 따라 RDB 파일 생성

### redis.conf

```
bind 0.0.0.0

port 6379

dbfilename backup.rdb

save 60 3

slave-read-only no
```

## RDB 파일로 데이터 복원

![alt text](20250206_164155.png)