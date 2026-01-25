
# 参数

username:


# 作用

1. 生成tokenId
2. 为token添加载荷（组织标签，和主组织标签）



```java
public String generateToken(String username) {  
    SecretKey key = getSigningKey(); // 解析密钥  
    // 获取用户信息  
    User user = userRepository.findByUsername(username)  
            .orElseThrow(() -> new RuntimeException("User not found"));  
      
    // 生成唯一的tokenId  
    String tokenId = generateTokenId();  
    long expireTime = System.currentTimeMillis() + EXPIRATION_TIME;  
      
    // 创建token内容  
    Map<String, Object> claims = new HashMap<>();  
    claims.put("tokenId", tokenId); // 添加tokenId用于Redis缓存  
    claims.put("role", user.getRole().name());  
    claims.put("userId", user.getId().toString()); // 添加用户ID到JWT  
    // 添加组织标签信息  
    if (user.getOrgTags() != null && !user.getOrgTags().isEmpty()) {  
        claims.put("orgTags", user.getOrgTags());  
    }  
      
    // 添加主组织标签信息  
    if (user.getPrimaryOrg() != null && !user.getPrimaryOrg().isEmpty()) {  
        claims.put("primaryOrg", user.getPrimaryOrg());  
    }  
  
    String token = Jwts.builder()  
            .setClaims(claims)  
            .setSubject(username)  
            .setExpiration(new Date(expireTime))  
            .signWith(key, SignatureAlgorithm.HS256)  
            .compact();  
      
    // 缓存token信息到Redis  
    tokenCacheService.cacheToken(tokenId, user.getId().toString(), username, expireTime);  
      
    logger.info("Token generated and cached for user: {}, tokenId: {}", username, tokenId);  
    return token;  
}
```