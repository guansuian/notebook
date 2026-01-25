# 参数

`tokenId:` token的id
`userId:` 用户的id（在集合中形成 用户 1-> n 设备token）
`usename:` 用户名字
`expireTimeMs:` 过期时间

# 作用

1. **将token录入白名单**
  ` redisTemplate.opsForValue().set(key, tokenInfo, ttlSeconds, TimeUnit.SECONDS);  `
  
2. 使用户其他设备
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


## set方法解析

`redisTemplate.opsForValue().set(key, tokenInfo, ttlSeconds, TimeUnit.SECONDS);`

这个方法在redis源码中长
`void set(K key, V value, long timeout, TimeUnit unit);`
这样

含义：设置键为`key`，值为`value`，并在`timeout` `unit` 后 (这个unit是时间单位，有可能是秒，也有可能是分钟)过期