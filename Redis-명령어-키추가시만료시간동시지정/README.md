# Redis 명령어 키 추가 시 만료 시간 동시 지정 (SETEX)

```
KEYS *
SETEX myKey1 10 "Hello World"
TTL myKey1
KEYS *
```

![alt text](20250206_145829.png)