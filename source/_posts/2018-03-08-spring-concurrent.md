---
title: springboot 并发处理
date: 2018-03-08 17:52:17
tags: java
---

## 使用 lock解决并发问题

springmvc默认是单例的线程不安全的，多个请求对应的是同一个对象，所以如果客户端并发访问特别多的时候，会造成代码线程安全。
比如说你代码里面可能会有if判断，但是如果线程不安全的 if判断不能保证了。
使用lock锁住可保持一致性

```java
class ControllerA {
private Lock insertLock = new ReentrantLock();

public void methodA() {
    try{
        insertLock.lock();
        doSomeThing();
    }finally{
        insertLock.unlock();
    }
    }
}

```

> ReentrantLock synchronized的对比可以网上查一下
> 另外还有reentrantreadwritelock啥的，等用的时候再说

## 使用redis解决并发

> 原理，利用setnx判断是存在key(setnx)来锁定，利用redis eval(lua)来原子性删除key(删除锁),部分代码如下

```
Redis自己实现下 ClusterRedis，和JedisPool
```

```java
@Component
public class RedisLockKit {
    private final Logger logger = LoggerFactory.getLogger(RedisLockKit.class);

    @Autowired
    private Redis redis;

    private ThreadLocal<String> lockFlag = new ThreadLocal<>();

    public static final String UNLOCK_LUA;

    static {
        StringBuilder sb = new StringBuilder();
        sb.append("if redis.call(\"get\",KEYS[1]) == ARGV[1] ");
        sb.append("then ");
        sb.append("    return redis.call(\"del\",KEYS[1]) ");
        sb.append("else ");
        sb.append("    return 0 ");
        sb.append("end ");
        UNLOCK_LUA = sb.toString();
    }


    /**
     * 加锁
     * @param key
     * @return
     */
    public boolean lock(String key, long expire) {
        boolean result = setRedis(key, expire);
        while((!result)){
            try {
                logger.debug("获取锁失败 重试中 {}...");
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                return false;
            }
            result = setRedis(key, expire);
        }
        return result;
    }

    /**
     * 解锁
     * @param key
     * @return
     */
    public boolean unLock(String key) {
        try {
            Long result = redis.eval(UNLOCK_LUA, Arrays.asList(key), Arrays.asList(lockFlag.get()));
            return result != null && result > 0;
        } catch (Exception e) {
            logger.error("解锁失败", e);
        }
        return false;
    }

    private boolean setRedis(String key, long expire) {
        try {
            String result = redis.process(commands-> {
                String uuid = UUID.randomUUID().toString();
                lockFlag.set(uuid);
                return commands.set(key, uuid, "NX", "PX", expire);
            });
            logger.info("result: {}",result);
            return !StringUtils.isEmpty(result);
        } catch (Exception e) {
            logger.error("key {} 设置失败", key, e);
        }
        return false;
    }
}
```