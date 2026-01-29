# 参数

`tokenId:` 短token的id

# 作用

检验前端传过来的短token是否有效
先是查询在黑名单是否有
然后再去白名单检查


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

## 我的思考

我觉得完全没有必要设置一个黑名单，只要白名单没有，就说明就是拉黑的token

所以这样修改

```java
/**
 * 验证token是否有效
 * 逻辑：只要 Redis 里能查到，且没过期，就是有效。
 * 如果被踢了，Redis 里自然查不到，直接返回 false。
 */
public boolean isTokenValid(String tokenId) {
    try {
        String key = TOKEN_PREFIX + tokenId;
        // 只需要这一步！
        // 如果管理员踢人执行了 DEL，这里直接返回 false，逻辑完美闭环。
        return Boolean.TRUE.equals(redisTemplate.hasKey(key));
    } catch (Exception e) {
        logger.error("Failed to check token validity: {}", tokenId, e);
        return false;
    }
}
```
