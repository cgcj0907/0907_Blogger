---
title: "Gin Middleware"
date: 2025-01-14
draft: false
tags: ["Languages", "Golang", "Gin", "Middleware"]
ShowToc: true
---

## 基本用法示例

```go
func Logger() gin.HandlerFunc {
  return func(c *gin.Context) {
    t := time.Now()

    // 设置 example 变量
    c.Set("example", "12345")

    // 请求前

    c.Next()

    // 请求后
    latency := time.Since(t)
    log.Print(latency)

    // 获取发送的 status
    status := c.Writer.Status()
    log.Println(status)
  }
}

func main() {
  r := gin.New()
  r.Use(Logger())

  r.GET("/test", func(c *gin.Context) {
    example := c.MustGet("example").(string)

    // 打印："12345"
    log.Println(example)
  })

  // 监听并在 0.0.0.0:8080 上启动服务
  r.Run(":8080")
}
```
## 常用函数（Gin Context）

> 本节中 `c` 均指 `c *gin.Context`

### HTTP 相关

1. 读取请求 Header

```go
c.GetHeader("example")
```

* 获取客户端请求携带的 Header
* 等价于 `c.Request.Header.Get("example")`
* 若 Header 不存在，返回空字符串
* 常用场景：

  * Authorization / Token
  * TraceID / RequestID
  * 客户端类型或版本信息

2. 写入请求 Header（网关 / 代理）

```go
c.Request.Header.Set("X-Request-ID", requestID)
```

* 修改即将转发给下游服务的请求 Header
* 用于链路追踪、灰度路由、用户身份传递等
* 在网关中常用于统一传递 Request ID 或 trace 信息

3. 写入响应 Header

```go
c.Writer.Header().Set("X-Request-ID", requestID)
```

* 设置当前响应返回给客户端的 Header
* 常用于：

  * 请求 ID
  * 服务版本
  * 链路追踪信息
* 注意：

  * 必须在响应体写出之前设置
  * 调用 `c.JSON` / `c.String` 后无法修改

### 上下文数据

1. 设置上下文数据

```go
c.Set("example", example)
```

* 请求级 Key-Value 存储
* 常用于：

  * Request ID
  * 用户信息
  * 鉴权结果
  * 中间计算状态
* 生命周期：仅在当前请求内有效

2. 获取上下文数据

```go
value, exists := c.Get("example")
```

* `exists` 用于判断 Key 是否存在，避免类型断言 panic

> 上下文数据是**请求内部共享状态**的主要机制，适合在中间件与 Handler 之间传递信息


### 控制流
1. c.Next()

```go
func Middleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 前置逻辑（请求进入时执行）

        c.Next() // 放行请求，执行后续中间件和 Handler

        // 后置逻辑（请求返回时执行）
    }
}
```

* 显式放行请求，执行后续中间件或 Handler
* `c.Next()` 之前：按注册顺序执行（FIFO）
* `c.Next()` 之后：按相反顺序回溯执行（LIFO）

2. c.Abort / c.AbortWithStatus

* 阻止后续中间件或 Handler 执行
* 常用于：

  * 鉴权失败
  * 参数校验异常
  * 请求异常返回

## 常见自定义中间件

> 鉴权相关中间件见👉 [Go Authorization](/languages/go/gin/authorization)
### Request ID 生成中间件
简单示例:
```go
func RequestIDMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		// TODO: Generate UUID for request ID
		// Use github.com/google/uuid package
		// Store in context as "request_id"
		// Add to response header as "X-Request-ID"
		requestID := uuid.New().String()
		c.Set("request_id", requestID)
		c.Writer.Header().Set("X-Request-ID", requestID)

		c.Next()
	}
}
```

### 日志记录中间件

简单示例:
```go
func LoggingMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // ==============================
        // 1. 捕获请求开始时间
        // ==============================
        startTime := time.Now()

        // ==============================
        // 2. 处理请求（调用后续中间件或路由）
        // ==============================
        c.Next()

        // ==============================
        // 3. 计算请求耗时
        // ==============================
        duration := time.Since(startTime)

        // ==============================
        // 4. 获取请求信息
        // ==============================
        requestID, _ := c.Get("request_id") // 可选，如果有中间件生成 request_id
        method := c.Request.Method
        path := c.Request.URL.Path
        status := c.Writer.Status()
        clientIP := c.ClientIP()
        userAgent := c.Request.UserAgent()

        // ==============================
        // 5. 格式化并打印日志
        // ==============================
        logLine := "[" + fmt.Sprintf("%v", requestID) + "] " +
            method + " " + path + " " +
            strconv.Itoa(status) + " " +
            duration.String() + " " +
            clientIP + " " +
            userAgent

        // 这里用 println 简单输出，也可以换成 logrus 或 zap 等日志库
        println(logLine)
    }
}
```

### CORS 中间件
简单示例:
```go
func CORSMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // ==============================
        // 1. 配置允许的来源、方法、请求头
        // ==============================
        allowedOrigins := []string{"http://localhost:3000", "https://myblog.com"}
        allowedMethods := "GET, POST, PUT, DELETE, OPTIONS"
        allowedHeaders := "Content-Type, X-API-Key, X-Request-ID"

        // 获取请求来源
        origin := c.GetHeader("Origin")

        // ==============================
        // 2. 设置 Access-Control-Allow-Origin
        // ==============================
        for _, o := range allowedOrigins {
            if origin == o {
                c.Writer.Header().Set("Access-Control-Allow-Origin", origin)
                break
            }
        }

        // ==============================
        // 3. 设置允许的方法和请求头
        // ==============================
        c.Writer.Header().Set("Access-Control-Allow-Methods", allowedMethods)
        c.Writer.Header().Set("Access-Control-Allow-Headers", allowedHeaders)

        // ==============================
        // 4. 处理预检请求（OPTIONS）
        // ==============================
        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(204) // No Content
            return
        }

        // ==============================
        // 5. 继续处理后续中间件或路由
        // ==============================
        c.Next()
    }
}
```

### 访问频率限制中间件

> 👉[相关库文档链接](https://pkg.go.dev/golang.org/x/time/rate)

简单示例:
```go
func RateLimiterMiddleware(next http.Handler) http.Handler {
    // 每个IP一个限流器
    clients := make(map[string]*rate.Limiter)
    mu := &sync.Mutex{}
    
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // 获取客户端IP（简化版，生产环境需要处理代理）
        ip := r.RemoteAddr
        
        mu.Lock()
        limiter, exists := clients[ip]
        
        // 如果不存在，创建新的限流器
        if !exists {
            // 每秒5个请求，突发10个
            limiter = rate.NewLimiter(5, 10)
            clients[ip] = limiter
        }
        mu.Unlock()
        
        // 检查是否允许请求
        if !limiter.Allow() {
            http.Error(w, "请求太频繁，请稍后再试", http.StatusTooManyRequests)
            return
        }
        
        next.ServeHTTP(w, r)
    })
}
```

#### 类型 `Limit`

```go
type Limit float64
```

**描述**：表示速率（单位：每秒允许事件数）
**辅助函数**：

```go
func Every(interval time.Duration) Limit
```

将“事件之间的时间间隔”转换为每秒允许事件数（即 `Limit`）


#### 类型 `Limiter`

```go
func NewLimiter(r Limit, b int) *Limiter
```

* `r`：每秒生成的 token 数（速率）
* `b`：token 桶的最大容量（突发容量，burst）

> Limiter 通过生成和消耗 token 来控制速率。消耗 token 的主要方法有：`Allow` / `AllowN`（立即判断） 、`Reserve` / `ReserveN`（预约） 、`Wait` / `WaitN`（阻塞等待）

`Allow`

```go
func (lim *Limiter) Allow() bool
```

**说明**：判断是否允许 1 个事件**立即**发生（非阻塞）
**用法场景**：要**直接丢弃/拒绝**超速事件的场景（例如 HTTP 返回 429）

---

`AllowN`

```go
func (lim *Limiter) AllowN(t time.Time, n int) bool
```

**说明**：判断在时间 `t` 是否可以立即发生 `n` 个事件（非阻塞）。
**用法场景**：一次性判断多个事件是否可立即执行

---

`Burst`

```go
func (lim *Limiter) Burst() int
```

**说明**：返回当前突发容量（桶大小 `b`），表示最大可累积的 token 数

---

`Limit`

```go
func (lim *Limiter) Limit() Limit
```

**说明**：返回当前速率 `Limit`（即创建或后续设置的 `r`）

---

`Reserve`

```go
func (lim *Limiter) Reserve() *Reservation
```

**说明**：等价于 `ReserveN(time.Now(), 1)`，返回一个 `Reservation`（预约 1 个事件）
**用法**：想要预约并稍后按速率执行（不丢弃事件）时使用

**示例**：

```go
r := lim.Reserve()
if !r.OK() { return }
time.Sleep(r.Delay())
Act()
```

---

`ReserveN`

```go
func (lim *Limiter) ReserveN(t time.Time, n int) *Reservation
```

**说明**：返回一个 `Reservation`，指示在多久后 `n` 个事件可以发生

* `r.OK()` 在 `n` 超过 `Burst()` 时返回 `false`
  **用法选择**：
* 想**等待并按速率慢慢执行**（不丢弃）→ `ReserveN`
* 想**丢弃超速事件** → `Allow/AllowN`
* 需要**上下文取消或超时** → `Wait/WaitN`

**示例**：

```go
r := lim.ReserveN(time.Now(), 1)
if !r.OK() { return }
time.Sleep(r.Delay())
Act()
```

---

`SetBurst`

```go
func (lim *Limiter) SetBurst(newBurst int)
```

**说明**：设置新的突发容量（等同于 `SetBurstAt(time.Now(), newBurst)`）
**注意**：对已预约但未执行的 `Reservation` 可能没有即时影响

---

`SetBurstAt`

```go
func (lim *Limiter) SetBurstAt(t time.Time, newBurst int)
```

**说明**：在指定时间 `t` 设置新的突发容量
**注意**：已预订但未执行的预约可能不会受到新设置的限制影响

---

`SetLimit`

```go
func (lim *Limiter) SetLimit(newLimit Limit)
```

**说明**：设置新的速率限制（等同于 `SetLimitAt(time.Now(), newLimit)`）
**注意**：更改速率不会回溯影响已经预约但未执行的操作

---

`SetLimitAt`

```go
func (lim *Limiter) SetLimitAt(t time.Time, newLimit Limit)
```

**说明**：在指定时间 `t` 设置新的速率限制
**注意**：同上，已预约的操作可能仍按旧规则保留其许可

---

`Tokens`

```go
func (lim *Limiter) Tokens() float64
```

**说明**：返回当前**可用的 token 数量**（可用于计算 `X-RateLimit-Remaining`）
**用法**：用于监控或在响应头中报告剩余配额

---

`TokensAt`

```go
func (lim *Limiter) TokensAt(t time.Time) float64
```

**说明**：返回在时间 `t` 时刻可用的 token 数（用于模拟未来状态）

---

`Wait`

```go
func (lim *Limiter) Wait(ctx context.Context) error
```

**说明**：等同于 `WaitN(ctx, 1)`，阻塞直到允许 1 个事件或上下文取消/超时
**用法**：需要阻塞并尊重上下文（可取消/超时）的场景

---

`WaitN`

```go
func (lim *Limiter) WaitN(ctx context.Context, n int) error
```

**说明**：阻塞直到允许 `n` 个事件或返回错误。错误条件包括：`n` 超过 `Burst()`、上下文被取消、等待超出上下文 Deadline（当速率为 `Inf` 时忽略 burst 限制）
**用法**：当必须按速率执行且需要可取消/超时控制时使用


#### 类型 `Reservation`

`Reservation` 表示 Limiter 为将来某一时刻保留的事件许可，包含是否可行、需要等待多长时间等信息。预约会影响后续许可判断

---

`Cancel`

```go
func (r *Reservation) Cancel()
```

**说明**：取消预约，等同于 `CancelAt(time.Now())`，尽量把所占用的 token 归还给 Limiter（不会破坏其他预约）

---

`CancelAt`

```go
func (r *Reservation) CancelAt(t time.Time)
```

**说明**：在指定时间 `t` 取消预约，尽量恢复预约对限流器的影响（按时间考虑其他预约的存在）

---

`Delay`

```go
func (r *Reservation) Delay() time.Duration
```

**说明**：返回需要等待的延迟时间，等同于 `DelayFrom(time.Now())`

---

`DelayFrom`

```go
func (r *Reservation) DelayFrom(t time.Time) time.Duration
```

**说明**：返回从时间 `t` 开始需要等待的时长；若等待超过最大允许等待时间则返回 `InfDuration`

---

`OK`

```go
func (r *Reservation) OK() bool
```

**说明**：指示在最大等待时间之前，是否能为所需事件提供足够 token（即预约是否有效）

### Content-Type 校验中间件

简单示例
```go
func ContentTypeMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {

		if c.Request.Method == "POST" || c.Request.Method == "PUT" {
			contentType := c.GetHeader("Content-Type")
			if contentType != "application/json" {
				c.AbortWithStatusJSON(http.StatusUnsupportedMediaType, gin.H{
					"error": "Unsupported Media Type",
				})
				return
			}
		}
		c.Next()
	}
}
```

### 错误处理中间件

简单示例:
```go
func ErrorHandlerMiddleware() gin.HandlerFunc {
	return gin.CustomRecovery(func(c *gin.Context, recovered interface{}) {

		requestID, _ := c.Get("request_id")
		c.JSON(http.StatusInternalServerError, gin.H{
			"error":      "Internal server error",
            "message":  fmt.Sprint(recovered),
			"request_id": requestID,
		})

	})
}
```

