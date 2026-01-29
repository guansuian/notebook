```java
public String extractUserIdFromToken(String token) {  
    try {  
        Claims claims = extractClaimsIgnoreExpiration(token);  
        return claims != null ? claims.get("userId", String.class) : null;  
    } catch (Exception e) {  
        logger.error("Error extracting userId from token: {}", token, e);  
        return null;  
    }  
}
```