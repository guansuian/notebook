
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