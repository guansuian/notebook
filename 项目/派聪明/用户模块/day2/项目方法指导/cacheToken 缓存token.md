# 参数

`tokenId:` token的id
`userId:` 用户的id（在集合中形成 用户 1-> n 设备token）
`usename:` 用户名字
`expireTimeMs:` 过期时间

# 作用

1. **将token录入白名单**
  ` redisTemplate.opsForValue().set(key, tokenInfo, ttlSeconds, TimeUnit.SECONDS);  `
  
2. **使用户其他设备上的`token`快速失效**
   假设用户有多个设备都有登录，为了能够同时将这个用户所有的token都失效，那么就将集合中的token全部删除 ,所以需要事先将用户的token加入到集合中
    `addTokenToUser(userId, tokenId, expireTimeMs);  `

```java
private static final String TOKEN_PREFIX = "jwt:valid:";

public void cacheToken(String tokenId, String userId, String username, long expireTimeMs) {  
    try {  
        String key = TOKEN_PREFIX + tokenId;  
        Map<String, Object> tokenInfo = new HashMap<>();  
        tokenInfo.put("userId", userId);  
        tokenInfo.put("username", username);  
        tokenInfo.put("expireTime", expireTimeMs);  
          
        // 计算Redis过期时间（比JWT过期时间稍长一点）  
        long ttlSeconds = (expireTimeMs - System.currentTimeMillis()) / 1000 + 300; // 多5分钟缓冲  
        redisTemplate.opsForValue().set(key, tokenInfo, ttlSeconds, TimeUnit.SECONDS);  
          
        // 同时添加到用户token集合中  
        addTokenToUser(userId, tokenId, expireTimeMs);  
          
        logger.debug("Token cached: {} for user: {}", tokenId, username);  
    } catch (Exception e) {  
        logger.error("Failed to cache token: {}", tokenId, e);  
    }  
}
```



## addTokenToUser方法

### 参数

`userId:` 用户id
`tokenId:` tokenId
`expireTimeMs:` 过期时间

### 作用

这个 addTokenToUser 方法的作用是 “建立用户到 Token 的反向索引” 。
简单来说，就是把这个新生成的 tokenId 记在 userId 的名下。

Redis 里的数据默认是 Key-Value 结构的，Key 通常是 tokenId 。

- 正向查询 ：有了 tokenId -> 查用户是谁。 (Redis 已经支持)
- 反向查询 ：有了 userId -> 查他有哪些 tokenId 。 (Redis 默认不支持，需要自己维护)
这个方法就是为了支持反向查询。

### 场景

场景：一键踢人 / 修改密码
- 当管理员要封禁张三 ( userId=1001 )，或者张三改了密码需要强制下线所有设备。
- 后端系统手里只有 userId=1001 。
- 通过这个方法建立的索引，后端可以去 Redis 查 jwt:user:1001:tokens ，瞬间得到一个列表 ( token_A, token_B, token_C ) 。
- 然后后端就可以遍历这个列表，把这 3 个 Token 全部删掉。
- 结果 ：张三的手机、电脑、iPad 全部同时下线。


```java
// 用户维度token集合键前缀，用于记录某用户下所有活跃token，便于批量登出
private static final String USER_TOKENS_PREFIX = "jwt:user:";

private void addTokenToUser(String userId, String tokenId, long expireTimeMs) {  
    try {  
        String key = USER_TOKENS_PREFIX + userId + ":tokens";  
        redisTemplate.opsForSet().add(key, tokenId);  
          
        // 设置过期时间  
        long ttlSeconds = (expireTimeMs - System.currentTimeMillis()) / 1000 + 300;  
        redisTemplate.expire(key, Duration.ofSeconds(ttlSeconds));  
    } catch (Exception e) {  
        logger.error("Failed to add token to user set: {} - {}", userId, tokenId, e);  
    }  
}
```


## set方法解析

`redisTemplate.opsForValue().set(key, tokenInfo, ttlSeconds, TimeUnit.SECONDS);`

这个方法在redis源码中长
`void set(K key, V value, long timeout, TimeUnit unit);`
这样

含义：设置键为`key`，值为`value`，并在`timeout` `unit` 后 (这个unit是时间单位，有可能是秒，也有可能是分钟)过期