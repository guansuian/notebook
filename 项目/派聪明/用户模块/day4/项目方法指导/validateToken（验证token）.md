# 参数

`token:` 前端发过来的token

# 作用

1. 先去redis里面检查
2. 再通过jwt进行验证

这个方法只返回false或者是true

```java
/**  
 * 验证 JWT Token 是否有效（优先使用Redis缓存）  
 */  
public boolean validateToken(String token) {  
    try {  
        // 首先从JWT中提取tokenId，若缺失则立即返回失败，避免后续无意义的Redis查询与签名验证  
        String tokenId = extractTokenIdFromToken(token);  
        if (tokenId == null) {  
            logger.warn("Token does not contain tokenId");  
            return false;  
        }  
          
        // 检查Redis缓存中的token状态  
        if (!tokenCacheService.isTokenValid(tokenId)) {  
            logger.debug("Token invalid in cache: {}", tokenId);  
            return false;  
        }  
          
        // Redis验证通过，再验证JWT签名（双重验证）  
        Jwts.parserBuilder()  
                .setSigningKey(getSigningKey())  
                .build()  
                .parseClaimsJws(token);  
  
        logger.debug("Token validation successful: {}", tokenId);  
        return true;  
    } catch (ExpiredJwtException e) {  
        logger.warn("Token expired: {}", e.getClaims().get("tokenId", String.class));  
    } catch (SignatureException e) {  
        logger.warn("Invalid token signature");  
    } catch (Exception e) {  
        logger.error("Error validating token", e);  
    }  
    return false;  
}
```
