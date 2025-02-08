# Redis Transaction

## 특징

- 트랜잭션 내에서 데이터 변경 여부를 감지해서, 다른 클라이언트가 데이터를 변경하면 이를 감지해서 현 트랜잭션을 실패시키는 방식이다.
- 트랜잭션 중에 호출되는 커맨드들은 큐에 보관 했다가 일시에 실행된다.
- Redis 트랜잭션은 롤백을 지원하지 않기 때문에 정합성이 중요한 데이터 처리는 RDBMS의 트랜잭션을 이용하는걸 권장한다.

## 트랜잭션 성공 케이스

- MULTI 호출로 트랜잭션이 시작된다.
- WATCH를 통해 KEY를 감시하고, 감시 도중 KEY의 값에 변동이 생기면 해당 트랜잭션은 무효 처리된다.
- 호출되는 커맨드들은 큐에다가 담아둔다.
- EXEC 호출로 큐에다 담아놓은 커맨드들이 실행되고 마무리가 되면 트랜잭션이 종료된다.

``` java
public List<Object> transferBalance(String key, Integer value) {
    return redisTemplate.execute(new SessionCallback<List<Object>>() {
        @Override
        public List<Object> execute(RedisOperations operations) throws DataAccessException {
            List<Object> result = null;

            try {
                operations.watch(key);
                operations.multi();
                
                if (value > 0) {
                    operations.opsForValue().increment(key, value);
                } else {
                    operations.opsForValue().decrement(key, -value);
                }
                result = operations.exec();
            } catch (Exception exception) {
                operations.discard();
            }

            return result;
        }
    });
}
```

## 트랜잭션 실패 케이스

- MULTI 호출로 트랜잭션이 시작된다.
- 호출되는 커맨드들은 큐에다가 담아둔다.
- DISCARD 호출로 큐에다 담아놓은 커맨드들은 실행되지 않고 트랜잭션이 종료된다.

``` java
public List<Object> discardBalance(String key, Integer value) {
    return redisTemplate.execute(new SessionCallback<List<Object>>() {
        @Override
        public List<Object> execute(RedisOperations operations) throws DataAccessException {
            List<Object> result = null;

            try {
                operations.watch(key);
                operations.multi();
                
                throw new RuntimeException("exception..");
            } catch (Exception exception) {
                log.error(exception.getMessage());
                operations.discard();
            }

            return result;
        }
    });
}
```

## 커맨드 에러 케이스

- MULTI 호출로 트랜잭션이 시작된다.
- 호출되는 커맨드들은 큐에다가 담아둔다. (커맨드가 유효하지 않은지 여부는 현 시점에선 중요하지 않다.)
- DISCARD 호출로 큐에다 담아놓은 커맨드들은 실행되지 않고 트랜잭션이 종료된다.
- EXEC 호출로 큐에다 담아놓은 커맨드들이 실행되고 유효한 커맨드들은 정상적을 DB에 저장된다.
- 유효하지 않은 커맨드가 실행되더라도 별다른 에러나 롤백은 이루어지지 않고 정상적으로 트랜잭션이 종료된다.

``` java
public void testTransactionRollback() {
    redisTemplate.execute((RedisConnection connection) -> {
        try {
            StringRedisConnection stringRedisConnection = (StringRedisConnection) connection;
            stringRedisConnection.multi();
            stringRedisConnection.set("key1", "value1");
            stringRedisConnection.expire("key1", -1);
            stringRedisConnection.set("key2", "value2");
            stringRedisConnection.exec();
        } catch (Exception e) {
            log.error(e.getMessage());
            connection.discard();
        }
        
        return null;
    });
}
```