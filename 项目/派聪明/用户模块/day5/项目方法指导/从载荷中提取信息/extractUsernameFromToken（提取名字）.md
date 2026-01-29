
[extractClaimsIgnoreExpiration（忽略token过期异常，提取载荷）](../../../day4/项目方法指导/extractClaimsIgnoreExpiration（忽略token过期异常，提取载荷）.md)
```java
public String extractUsernameFromToken(String token) {  
    try {  
        Claims claims = extractClaimsIgnoreExpiration(token);  
        return claims != null ? claims.getSubject() : null;  
    } catch (Exception e) {  
        logger.error("Error extracting username from token: {}", token, e);  
        return null;  
    }  
}
```