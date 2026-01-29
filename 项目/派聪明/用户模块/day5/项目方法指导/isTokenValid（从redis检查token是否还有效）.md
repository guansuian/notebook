# 参数

`tokenId:` 短token的id

# 作用

检验前端传过来的短token是否有效
zai



```java
// 有效JWT token缓存键前缀，用于存储当前未过期的token完整信息  
private static final String TOKEN_PREFIX = "jwt:valid:";
/**  
 * 验证token是否有效（未被拉黑且存在于缓存中）  
 */  
public boolean isTokenValid(String tokenId) {  
    try {  
        // 先检查是否在黑名单中  
        if (isTokenBlacklisted(tokenId)) {  
            return false;  
        }  
          
        // 检查缓存中是否存在  
        String key = TOKEN_PREFIX + tokenId;  
        return Boolean.TRUE.equals(redisTemplate.hasKey(key));  
    } catch (Exception e) {  
        logger.error("Failed to check token validity: {}", tokenId, e);  
        return false;  
    }  
}
```