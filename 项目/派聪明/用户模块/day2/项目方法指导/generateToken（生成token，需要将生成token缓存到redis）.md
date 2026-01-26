
# 参数

`username:` 用户的名字 用于查询


# 作用

1. 生成tokenId
2. 为token添加载荷（组织标签，和主组织标签）

```java
//这个就是短token
private static final long EXPIRATION_TIME = 3600000; // 1 hour (调整为1小时)
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

## 存在token里面的载荷和存在redis里面的参数的差别

|       | tokenId                 | username        | expireTime                    | userId         |
| ----- | ----------------------- | --------------- | ----------------------------- | -------------- |
| 载荷    | 有                       | 有               | 有                             | 有              |
| redis | 有（作为键）被存放，即一个短token就是一行 | 有（作为值被存放在redis） | 有（即存放在redis里面，又作为token过期的定时器） | 有（作为值存入redis中） |

### redis里面 token存在集合和


[cacheToken 缓存token](cacheToken%20缓存token.md)