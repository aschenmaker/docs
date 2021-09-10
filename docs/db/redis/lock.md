# redis 实现分布式锁

!!! note "分布式锁"
		为了确保服务的无状态，需要将状态信息，进行共享。Redis可以完成这件事情。但是由于服务的并发访问，我们需要维护数据的一致性，实现分布式锁就很关键。本文介绍了如何实现分布式锁，以及分析了分布式锁常见的问题。
		
    本文转载自 `小米系信息部技术团队`[^1]




[^1]: 参考原文：https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/
		


## 1. intro

在分布式系统中，为了维护服务状态信息，我们需要把一些数据放置在redis中进行共享和同步，这就会导致一个问题，如果分布式系统中的多个服务，同时对数据进行修改，无法确保数据的一致性。

因此我们需要维护一个分布式锁来保证数据的一致性。

![redis-lock-01](https://dist.lyneee.com/blog/2021-09-11-redis-lock-01.png)

## 2.Implement

Redis锁需要利用到Redis的 `setnx`命令，set if not exist。

- 加锁：`SETNX key value`，当键不存在时，对键进行设置操作并返回成功，否则返回失败。KEY 是锁的唯一标识，一般按业务来决定命名。
- 释放锁：`DEL key`，通过删除键值对释放锁，以便其他线程可以通过 SETNX 命令来获取锁。
- 锁超时：`EXPIRE key timeout`, 设置 key 的超时时间，以保证即使锁没有被显式释放，锁也可以在一定时间后自动释放，避免资源被永远锁住。

```go
func sync(){
  if setnx(key, 1) == 1{
    expire(key, 30s)
    ...
    do()
    ...
    del(key)
  }
}
```

但是这样的方式存在一定的问题：

### 2.1 SETNX 和 EXPIRE 非原子性

如果设置锁成功，但是设置超时时间失败（可能是网络、或者服务器故障导致），锁无法被释放，变成死锁。

![lock-2](https://dist.lyneee.com/blog/2021-09-11-redis-lock-02.png)

所以，需要确保操作时原子性的。使用 lua 脚本来确保原子性。

```lua
if (redis.call('setnx', KEYS[1], ARGV[1]) < 1)
then return 0;
end;
redis.call('expire', KEYS[1], tonumber(ARGV[2]));
return 1;

// 使用实例
EVAL "if (redis.call('setnx',KEYS[1],ARGV[1]) < 1) then return 0; end; redis.call('expire',KEYS[1],tonumber(ARGV[2])); return 1;" 1 key value 100

```

### 2.2 锁误接触

如果线程 A 成功获取到了锁，并且设置了过期时间 30 秒，但线程 A 执行时间超过了 30 秒，锁过期自动释放，此时线程 B 获取到了锁；随后 A 执行完成，线程 A 使用 DEL 命令来释放锁，但此时线程 B 加的锁还没有执行完成，线程 A 实际释放的线程 B 加的锁。

![lock-3](https://dist.lyneee.com/blog/2021-09-11-redis-lock-03.png)

通过在 value 中设置当前线程加锁的标识，在删除之前验证 key 对应的 value 判断锁是否是当前线程持有。可生成一个 UUID 标识当前线程，使用 lua 脚本做验证标识和解锁操作。

```lua
// 加锁
String uuid = UUID.randomUUID().toString().replaceAll("-","");
SET key uuid NX EX 30
// 解锁
if (redis.call('get', KEYS[1]) == ARGV[1])
    then return redis.call('del', KEYS[1])
else return 0
end
```

### 2.3 超时解锁导致并发

如果线程 A 成功获取锁并设置过期时间 30 秒，但线程 A 执行时间超过了 30 秒，锁过期自动释放，此时线程 B 获取到了锁，线程 A 和线程 B 并发执行。

![lock-4](https://dist.lyneee.com/blog/2021-09-11-redis-lock-04.png)

A、B 两个线程发生并发显然是不被允许的，一般有两种方式解决该问题：

- 将过期时间设置足够长，确保代码逻辑在锁释放之前能够执行完成。
- 为获取锁的线程增加守护线程，为将要过期但未释放的锁增加有效时间。

![lock-5](https://dist.lyneee.com/blog/2021-09-11-redis-lock-05.png)

### 2.4  不可重入

当线程在持有锁的情况下再次请求加锁，如果一个锁支持一个线程多次加锁，那么这个锁就是可重入的。如果一个不可重入锁被再次加锁，由于该锁已经被持有，再次加锁会失败。Redis 可通过对锁进行重入计数，加锁时加 1，解锁时减 1，当计数归 0 时释放锁。

### 2.5 无法等待-锁释放

如果服务器无法等待锁释放的问题通过两个方式来解决：

- 服务器对redis进行轮询，直到能够获得锁。但是这样做的缺点是，轮询导致服务器资源消耗，并发量大的时候，影响服务器效率
- 利用Redis发布订阅的功能，如果获取锁失败后，对锁消息进行订阅。

![lock-06](https://dist.lyneee.com/blog/2021-09-11-redis-lock-06.png)

