# Redis 데이터 타입 Hash 처리

HASH 데이터 타입은 하나의 키 아래 여러 개의 필드-값을 저장할 수 있다.

## 데이터 추가

```
HSET user name "KIM" age 29 country "KR"
HGET user name
HGET user age
HGET user country
```

![alt text](20250129_232618.png)

## 데이터 삭제

```
HSET user name "KIM" age 29 country "KR"
HLEN user
HDEL user name
HLEN user
HDEL user age
HLEN user
HDEL user country
HLEN user
```

![alt text](20250129_234253.png)

## 데이터 조회

```
HSET user name "KIM" age 29 country "KR"

HGET user name
HGET user age
HGET user country

HGETALL user

HKEYS user

HVALS user
```

![alt text](20250129_232921.png)

## 필드 개수 조회

```
HLEN user
HSET user name "KIM"
HLEN user
HSET user age 29
HLEN user
HSET user country "KR"
HLEN user
```

![alt text](20250129_233027.png)

## 필드 존재 여부 확인

```
HSET user name "KIM"
HEXISTS user name
HEXISTS user age
```

![alt text](20250129_234151.png)