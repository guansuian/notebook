# 参数

```java
username: 用户账号
password: 密码
record UserRequest(String username, String password) {}
```

# 作用

1. **验证用户是否登录** 

```java
@PostMapping("/login")  
public ResponseEntity<?> login(@RequestBody UserRequest request) {  
    LogUtils.PerformanceMonitor monitor = LogUtils.startPerformanceMonitor("USER_LOGIN");  
    try {  
        if (request.username() == null || request.username().isEmpty() ||  
                request.password() == null || request.password().isEmpty()) {  
            LogUtils.logUserOperation("anonymous", "LOGIN", "validation", "FAILED_EMPTY_PARAMS");  
            return ResponseEntity.badRequest().body(Map.of("code", 400, "message", "Username and password cannot be empty"));  
        }  
          
        String username = userService.authenticateUser(request.username(), request.password());  
        if (username == null) {  
            LogUtils.logUserOperation(request.username(), "LOGIN", "authentication", "FAILED_INVALID_CREDENTIALS");  
            return ResponseEntity.status(401).body(Map.of("code", 401, "message", "Invalid credentials"));  
        }  
          
        String token = jwtUtils.generateToken(username);  
        String refreshToken = jwtUtils.generateRefreshToken(username);  
        LogUtils.logUserOperation(username, "LOGIN", "token_generation", "SUCCESS");  
        monitor.end("登录成功");  
          
        return ResponseEntity.ok(Map.of("code", 200, "message", "Login successful", "data", Map.of(  
            "token", token,  
            "refreshToken", refreshToken  
        )));  
    } catch (CustomException e) {  
        LogUtils.logBusinessError("USER_LOGIN", request.username(), "登录失败: %s", e, e.getMessage());  
        monitor.end("登录失败: " + e.getMessage());  
        return ResponseEntity.status(e.getStatus()).body(Map.of("code", e.getStatus().value(), "message", e.getMessage()));  
    } catch (Exception e) {  
        LogUtils.logBusinessError("USER_LOGIN", request.username(), "登录异常: %s", e, e.getMessage());  
        monitor.end("登录异常: " + e.getMessage());  
        return ResponseEntity.status(500).body(Map.of("code", 500, "message", "Internal server error"));  
    }  
}
```