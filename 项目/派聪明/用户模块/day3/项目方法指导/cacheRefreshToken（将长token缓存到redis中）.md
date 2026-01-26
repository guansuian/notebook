


```java
// refresh token缓存键前缀，用于存储refresh token及其关联的access token信息  
private static final String REFRESH_PREFIX = "jwt:refresh:";

public void cacheRefreshToken(String refreshTokenId, String userId, String tokenId, long expireTimeMs) {  
    try {  
        String key = REFRESH_PREFIX + refreshTokenId;  
        Map<String, Object> refreshInfo = new HashMap<>();  
        refreshInfo.put("userId", userId);  
        refreshInfo.put("tokenId", tokenId);  
        refreshInfo.put("expireTime", expireTimeMs);  
          
        long ttlSeconds = (expireTimeMs - System.currentTimeMillis()) / 1000;  
        redisTemplate.opsForValue().set(key, refreshInfo, ttlSeconds, TimeUnit.SECONDS);  
          
        logger.debug("Refresh token cached: {} for user: {}", refreshTokenId, userId);  
    } catch (Exception e) {  
        logger.error("Failed to cache refresh token: {}", refreshTokenId, e);  
    }  
}
```