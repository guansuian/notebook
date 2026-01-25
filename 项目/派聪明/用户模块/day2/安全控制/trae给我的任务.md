你想学习这套 **“双 Token + Redis 状态控制”** 的认证体系是非常明智的。这是后端开发面试中 **含金量极高** 的考点，也是从“CRUD 程序员”进阶到“架构师思维”的必经之路。

根据你的技术背景（Java 后端基础不错，追求企业级实战），我为你定制了这份 **循序渐进的学习路线**：

### 阶段一：基础原理攻坚 (The Why & The What)
**目标**：不看代码，先搞懂“为什么要这么做”。
1.  **JWT 结构**：彻底弄懂 Header、Payload、Signature 三部分。明白为什么 JWT 是“无状态”的。
2.  **Access vs Refresh**：
    *   **思考题**：如果只有 Access Token，设长了不安全，设短了用户烦，怎么解？（引出 Refresh Token）。
    *   **思考题**：Refresh Token 为什么比 Access Token 安全？（因为它只在刷新时传输一次，暴露面小；且通常配合数据库/Redis 校验）。

### 阶段二：代码实战拆解 (Code Deep Dive)
**目标**：结合本项目代码，看懂每一行逻辑。建议按以下顺序阅读：

1.  **Token 生成 (`JwtUtils.java`)**：
    *   重点看 `generateToken` vs `generateRefreshToken`。
    *   关注 `claims` 里放了什么？（UserId, Role, OrgTags）。**千万别放密码！**
    *   关注 Redis 是怎么介入的 (`tokenCacheService.cacheToken`)。

2.  **登录流程 (`UserController.java` -> `/login`)**：
    *   看它怎么校验密码，怎么生成双 Token，怎么返回给前端。

3.  **Token 校验 (`JwtUtils.java` -> `validateToken`)**：
    *   **核心逻辑**：不仅验签名（Crypto），还要去 Redis 查（Stateful）。这是本项目最精彩的地方。
    *   **思考**：如果 Redis 挂了，这套系统还能用吗？（答案：不能，这叫“强依赖”）。

4.  **拦截器/过滤器 (`JwtAuthenticationFilter.java`)**：
    *   看它是怎么从 HTTP Header 里抠出 `Bearer token` 的。
    *   看它是怎么把解析出来的 User 信息塞进 Spring Security 上下文的 (`SecurityContextHolder`)。

### 阶段三：进阶与面试防御 (Advanced & Interview)
**目标**：应对面试官的“刁难”。

1.  **无感知刷新 (Silent Refresh)**：
    *   **前端逻辑**：当前端发请求收到 `401 Token Expired` 时，如何**自动**拦截请求 -> 用 Refresh Token 换新票 -> 重发原请求？（这是前端难点，但后端要知道原理）。
    *   **后端支持**：后端需要提供 `/refresh-token` 接口（本项目有吗？去找找）。

2.  **黑名单机制 (Blacklist)**：
    *   **场景**：用户改密码了、手机丢了，如何让之前的 Token 立即失效？
    *   **本项目解法**：直接删 Redis。

3.  **并发刷新问题 (Race Condition)**：
    *   **场景**：页面并发发了 5 个请求，Token 刚好过期。5 个请求同时去刷新 Token，会导致 Refresh Token 瞬间失效（如果 Refresh Token 是一次性的）。
    *   **解法**：Redis 锁，或者允许 Refresh Token 在生成新 Token 后有一段“宽限期 (Grace Period)”。

### 学习任务清单 (Action List)

1.  **阅读**：`src/main/java/com/yizhaoqi/smartpai/utils/JwtUtils.java`（核心工具类）。
2.  **阅读**：`src/main/java/com/yizhaoqi/smartpai/service/TokenCacheService.java`（Redis 交互逻辑）。
3.  **寻找**：去 `UserController` 里找找有没有 `/refresh` 接口？如果没有，思考一下如果要加该怎么写？

**现在，请打开 `JwtUtils.java`，我们先从 Token 的生成逻辑看起。**