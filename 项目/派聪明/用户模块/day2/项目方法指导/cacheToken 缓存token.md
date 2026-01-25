# 参数

tokenId:
userId:
usename:
expireTimeMs:

# 作用


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
