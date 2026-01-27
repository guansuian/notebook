
# 参数

`token:` token的内容

# 作用

忽略过期异常，提取token中的载荷
能提取：就说明没有人篡改token

```java
/**  
 * 提取Claims，忽略过期异常  
 */  
private Claims extractClaimsIgnoreExpiration(String token) {  
    try {  
        return Jwts.parserBuilder()  
                .setSigningKey(getSigningKey())  
                .build()  
                .parseClaimsJws(token)  
                .getBody();  
    } catch (ExpiredJwtException e) {  
        // 忽略过期异常，返回claims  
        return e.getClaims();  
    } catch (Exception e) {  
        logger.debug("Cannot extract claims from token: {}", e.getMessage());  
        return null;  
    }  
}
```