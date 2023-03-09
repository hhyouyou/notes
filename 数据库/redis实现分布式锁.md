# Redis 实现分布式锁

1. 获取 Redis 连接：通过连接池获取 Redis 连接。
2. 设置锁：使用 setnx 命令来设置一个键值对，如果当前键不存在，则设置成功并返回成功标识，否则返回失败标识。
3. 设置过期时间：为了防止死锁，在获取到锁之后需要为该键值对设置一个过期时间，可以使用 expire 命令。
4. 释放锁：在获取到锁之后，需要在解锁时使用 del 命令将该键值对删除，从而释放锁。

以下是一个简单的 Redis 分布式锁的实现代码示例（使用 RedisTemplate）：

```
public class RedisDistributedLock {

    private RedisTemplate redisTemplate;

    private static final int DEFAULT_EXPIRE_TIME = 60; // 默认锁定时间为 60 秒

    public RedisDistributedLock(RedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    public boolean lock(String key) {
        return lock(key, DEFAULT_EXPIRE_TIME);
    }

    public boolean lock(String key, int expireTime) {
        Boolean result = redisTemplate.opsForValue().setIfAbsent(key, "locked");
        if (result != null && result) {
            redisTemplate.expire(key, expireTime, TimeUnit.SECONDS);
            return true;
        }
        return false;
    }

    public void unlock(String key) {
        redisTemplate.delete(key);
    }
}
```

这里我们通过 `redisTemplate` 对象进行操作，其中 `setIfAbsent` 方法相当于 `setnx` 命令，`expire` 方法相当于 `expire` 命令，`delete` 方法相当于 `del` 命令。在上面的代码中，我们提供了两个重载方法，其中一个使用默认的锁定时间，另一个可以指定过期时间。在调用 `lock()` 方法时，如果获取到了锁，则返回 `true`，否则返回 `false`。在调用 `unlock()` 方法时，将会删除该键值对，从而释放锁。