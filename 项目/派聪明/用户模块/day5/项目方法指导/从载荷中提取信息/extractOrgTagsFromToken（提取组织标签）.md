
[extractClaimsIgnoreExpiration（忽略token过期异常，提取载荷）](../../../day4/项目方法指导/extractClaimsIgnoreExpiration（忽略token过期异常，提取载荷）.md)
```java
/**  
 * 从 JWT Token 中提取组织标签  
 */  
public String extractOrgTagsFromToken(String token) {  
    try {  
        Claims claims = extractClaimsIgnoreExpiration(token);  
        return claims != null ? claims.get("orgTags", String.class) : null;  
    } catch (Exception e) {  
        logger.error("Error extracting organization tags from token: {}", token, e);  
        return null;  
    }  
}
```