# 参数

`username:` 用户的名字 用于查询

# 作用

1. 生成tokenId
2. 为token添加载荷（组织标签，和主组织标签）

```java
private static final long REFRESH_TOKEN_EXPIRATION_TIME = 604800000; // 7 days (refresh token有效期)

public String generateRefreshToken(String username) {  
    SecretKey key = getSigningKey();  
      
    // 获取用户信息  
    User user = userRepository.findByUsername(username)  
            .orElseThrow(() -> new RuntimeException("User not found"));  
      
    // 生成唯一的refreshTokenId  
    String refreshTokenId = generateTokenId();  
    long expireTime = System.currentTimeMillis() + REFRESH_TOKEN_EXPIRATION_TIME;  
      
    // 创建refreshToken内容（相对简单，只包含基本信息）  
    Map<String, Object> claims = new HashMap<>();  
    claims.put("refreshTokenId", refreshTokenId); // 添加refreshTokenId  
    claims.put("userId", user.getId().toString());  
    claims.put("type", "refresh"); // 标识这是一个refresh token  
  
    String refreshToken = Jwts.builder()  
            .setClaims(claims)  
            .setSubject(username)  
            .setExpiration(new Date(expireTime))  
            .signWith(key, SignatureAlgorithm.HS256)  
            .compact();  
      
    // 缓存refresh token信息到Redis  
    tokenCacheService.cacheRefreshToken(refreshTokenId, user.getId().toString(), null, expireTime);  
      
    logger.info("Refresh token generated and cached for user: {}, refreshTokenId: {}", username, refreshTokenId);  
    return refreshToken;  
}
```

## 存在token里面的载荷和存在redis里面的参数的差别

|       | tokenId                 | username        | expireTime                    | userId         |
| ----- | ----------------------- | --------------- | ----------------------------- | -------------- |
| 载荷    | 有                       | 有               | 有                             | 有              |
| redis | 有（作为键）被存放，即一个短token就是一行 | 有（作为值被存放在redis） | 有（即存放在redis里面，又作为token过期的定时器） | 有（作为值存入redis中） |

