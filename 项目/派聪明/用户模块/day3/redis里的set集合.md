这也是一个**如果不做，上线半年后服务器内存爆炸**的关键细节。

答案是：**必须设置，而且要有策略地设置。**

如果不设置过期时间，这些 Set 集合（`user:{uid}:tokens`）会成为 Redis 里的**“僵尸数据”**。

---

### 1. 为什么要设置？（后果演示）

假设你的产品火了，有 100 万注册用户。

- 其中 90 万人是“过客”，登录一次后，卸载了 App，再也没回来。
    
- **如果你不设置过期时间：** 这 90 万个 `user:{id}:tokens` 的集合会**永久**停留在 Redis 内存里。
    
- **后果：** 随着时间推移，Redis 内存被这些几百年前的废弃索引占满，导致新用户登录不进来，或者因为内存不足触发 Redis 的驱逐策略（Eviction），把有用的数据挤掉了。
    

---

### 2. 设置多久合适？

这个 Set 集合是用来存 Token 索引的，所以它的寿命应该**和你的长 Token (Refresh Token) 保持一致**。

- **原则：** `Set 的 TTL` >= `Refresh Token 的 TTL`。
    
- **建议：** 如果你的 Refresh Token 有效期是 7 天，那么这个 Set 的过期时间建议设置为 **7 天**（或者稍微多一点，比如 7 天 + 1 小时）。
    

---

### 3. 关键逻辑：自动续期 (Keep Alive)

这里有一个**非常容易写出的 Bug**，请务必注意：

你不能只在“注册/第一次登录”的时候设置过期时间。**每次用户有了新动作，你都要给这个 Set “续命”。**

#### 错误场景：

1. **Day 1:** 用户登录。Refresh Token 有效期 7 天。Set 有效期设置为 7 天。
    
2. **Day 6:** 用户活跃，触发刷新。生成了新的 Refresh Token（有效期到 Day 13）。
    
    - _Bug 点：_ 此时你往 Set 里加了新 ID，**但忘记重置 Set 的过期时间**。
        
3. **Day 7:** Set 集合因为 Day 1 设置的时间到了，**自动消失了**。
    
4. **Day 8:** 用户拿着新 Token 访问，一切正常（因为 Key 2 还在）。
    
5. **Day 8 (出事了):** 管理员想踢这个用户。代码去查 Set……**发现是空的！**
    
    - 系统误判：管理员以为该用户已经下线了。
        
    - 结果：踢人失败，用户继续逍遥法外。
        

#### 正确做法：续期代码模版

在你的 `AuthService` 或者 Redis 工具类里，封装一个 `addTokenToSet` 方法：

```java
public void addTokenToUserSet(Long userId, String tokenEntry) {
    String key = "user:" + userId + ":tokens";
    
    // 1. 存入数据
    redisTemplate.opsForSet().add(key, tokenEntry);
    
    // 2. 【核心】每次操作都重新设置过期时间！
    // 强制复活这个 Set，让它从现在开始再活 7 天
    redisTemplate.expire(key, 7, TimeUnit.DAYS); 
}
```

---

### 4. 那么，Set 里的“过期 ID”怎么办？

你可能会问：_“如果 Set 一直续命，那里面那些几年前的旧 Token ID 岂不是一直删不掉？”_

这就涉及到了**集合内部的清理**。虽然 Set 作为一个整体（Key）续期了，但里面的元素（Value）需要维护。

有两种策略：

**策略 A：懒人策略 (靠轮转自动删)**

只要你的“令牌轮转”逻辑写得对（每次生成新的，都 `SREM` 删掉旧的），那么 Set 里的元素数量应该非常稳定（比如永远只有 2-4 个）。你不需要额外担心垃圾堆积。

**策略 B：强迫症策略 (定时清理)**

在登录接口里，顺手做一个检查：


```java
// 获取 Set 里所有的 entry
Set<String> entries = redisTemplate.opsForSet().members(key);
for (String entry : entries) {
    // 检查每一个对应的 Key 还在不在
    // 比如 entry 是 "ACCESS:xyz"，去查 "jwt:valid:xyz"
    if (!redisTemplate.hasKey(convertToRealKey(entry))) {
        // 如果本体都没了，这个索引留着也没用，删掉
        redisTemplate.opsForSet().remove(key, entry);
    }
}
```

_注：策略 B 会增加 Redis 的读写压力，除非你的 Set 异常膨胀，否则通常不需要这样做。**策略 A + 设置 Set 整体过期时间** 已经足够应付 99% 的场景。_

### 总结

1. **必须设过期时间**：否则 Redis 变成垃圾场。
    
2. **时长**：等于 Refresh Token 的时长（如 7 天）。
    
3. **动作**：每次 `SADD` (添加新 Token) 时，必须顺手 `EXPIRE` (重置 7 天倒计时)。