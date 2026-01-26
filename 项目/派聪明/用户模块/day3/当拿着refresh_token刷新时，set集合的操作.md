这三个场景总结得非常精准！这涵盖了 Refresh Token 所有的生命周期状态。

下面我针对这三种情况，详细拆解 **Set 集合 (`user:{uid}:tokens`)** 究竟发生了什么操作。

为了方便记忆，我给它们起了三个代号：**静默（无视）**、**毁灭（清空）**、**更替（轮转）**。

---

### 场景 1：已经过期 (Natural Expiration)

**代号：静默 (Ignored)**

用户很久没登录，Refresh Token 的 7 天有效期到了。前端发来请求，但 Token 里的时间戳 (`exp`) 已经是过去式。

- **前提：** JWT 解析库（如 `jjwt`）在校验签名和时间时，直接抛出 `ExpiredJwtException` 异常。
    
- **代码逻辑：** 代码在第一行校验时就报错中断了，根本没有走到连接 Redis 那一步。
    
- **Set 集合操作：** **无操作 (No Operation)**。
    
    - Redis 里的 Set 集合保持原样，什么都没动。
        
    - _注意：_ 此时 Set 里其实残留着这个过期的 ID（变成了僵尸数据）。这通常依靠我们之前提到的“惰性删除”策略（下次查到它是废的时顺手删掉），或者设置 Redis Key 过期监听来清理。但在**当前这次请求**中，Set 是完全没被触碰的。
        
- **结果：** 返回 401，用户被踢去登录页。
    

---

### 场景 2：重放/攻击 (Replay Attack)

**代号：毁灭 (Destruction)**

这是最危急的情况。JWT 没过期（时间是对的），签名也没错（是系统发的），**但是**去 Redis 查这个 Refresh Token 的 Key (`jwt:refresh:xxx`) 竟然**不存在**！

这说明这个 Token 已经被“轮转”过（被销毁了），现在是有人拿着废票想混进来。

- **前提：** `Signature` = OK, `Expiration` = OK, `Redis Key` = NULL。
    
- **Set 集合操作：** **删除整个 Key (DEL)**。
    
    - **命令：** `DEL user:{uid}:tokens`
        
    - **含义：** 我不光要拒绝你这次请求，我还要把你这个用户在**所有设备**上的所有记录（Set 里的所有 ID）全部抹除。
        
    - _效果：_ 这个 Set 集合瞬间消失。
        
- **关联操作：**
    
    - 通常还会顺手记录一条安全日志。
        
- **结果：** 返回 401/403，强制用户重新登录，且其他设备也全部掉线。
    

---

### 场景 3：正常刷新 (Happy Path)

**代号：更替 (Rotation)**

这是最常见的场景。用户正常使用，Token 快过期了，来换个新的。JWT 没过期，Redis 里也能查到这个 Refresh Token。

- **前提：** `Signature` = OK, `Expiration` = OK, `Redis Key` = Exists。
    
- **Set 集合操作：** **先剔除旧的，再加入新的 (SREM + SADD)**。
    
    **步骤 A：剔除旧数据**
    
    - **命令：** `SREM user:{uid}:tokens {old_short_id} {old_long_id}`
        
    - _含义：_ 把这次请求带来的、马上要作废的两个旧 ID 从集合里抠出去。
        
    
    **步骤 B：加入新数据**
    
    - **命令：** `SADD user:{uid}:tokens {new_short_id} {new_long_id}`
        
    - _含义：_ 把刚刚生成好的、热乎的两个新 ID 加进集合里。
        
- **结果：** 返回新的 Access + Refresh Token，用户无感续期。
    

---

### 三种情况对比表

|**场景**|**JWT 状态**|**Redis 查 Refresh Key 结果**|**Set 集合 (user:uid:tokens) 操作**|**业务含义**|
|---|---|---|---|---|
|**1. 已过期**|**Expired** (挂了)|(未执行)|**无操作** (代码提前抛异常中断)|正常的老死，请重新登录。|
|**2. 攻击/重放**|Valid (活着)|**NULL** (没查到)|**`DEL` (直接删除整个集合)**|有内鬼！全部封杀，强制下线。|
|**3. 正常刷新**|Valid (活着)|**Exists** (查到了)|**`SREM` 旧 ID + `SADD` 新 ID**|以旧换新，继续服务。|

### 导师点评

你在设计代码时，一定要把 **场景 1** 放在最前面。

Java

```java
try {
    // 1. JWT 校验 (库函数自带)
    jwtUtil.parse(token); 
} catch (ExpiredJwtException e) {
    // 对应场景 1：直接返回过期，千万别去查 Redis，浪费性能
    throw new UnauthException("已过期");
}

// 2. 查 Redis
if (!redis.hasKey(refreshKey)) {
    // 对应场景 2：没过期但 Redis 里没有 -> 攻击！
    redis.delete(userSetKey); // 删 Set
    throw new SecurityException("令牌被盗用");
}

// 3. 对应场景 3：正常轮转
redis.opsForSet().remove(userSetKey, oldIds); // 删 Set 里的旧
redis.opsForSet().add(userSetKey, newIds);    // 加 Set 里的新
```
