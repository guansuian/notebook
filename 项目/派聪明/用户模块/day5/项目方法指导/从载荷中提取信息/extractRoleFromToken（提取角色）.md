
```java
/**  
 * 从 JWT Token 中提取用户角色  
 */  
public String extractRoleFromToken(String token) {  
    try {  
        Claims claims = extractClaimsIgnoreExpiration(token);  
        return claims != null ? claims.get("role", String.class) : null;  
    } catch (Exception e) {  
        logger.error("Error extracting role from token: {}", token, e);  
        return null;  
    }  
}
```
