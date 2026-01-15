---
title: "Gin QuickStart"
date: 2025-01-07
draft: false
tags: ["Languages", "Golang", "Gin"]
ShowToc: true
---

## 基本用法示例

```go
package main

import (
  "github.com/gin-gonic/gin"
  "net/http"
)

func main() {
  router := gin.Default()
  router.GET("/ping", func(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
      "message": "pong",
    })
  })
  router.Run()
}
```
* 一般情况下 Gin 和 go 的 HTTP 模块配合使用, Gin 负责基本框架, go 的 HTTP 模块负责状态码管理等

* 默认路由为 `router := gin.Default`
* 启动命令为 `router.Run()`, 默认监听 8000 端口

### 自定义配置
```go
// 不指定 Gin 的绑定地址和端口。默认绑定所有接口，端口为 8080。
// 使用不带参数的 `Run()` 时，可通过 `PORT` 环境变量更改监听端口。
router := gin.Default()
router.Run()

// 指定 Gin 的绑定地址和端口。
router := gin.Default()
router.Run("192.168.1.100:8080")

// 仅指定监听端口。将绑定所有接口。
router := gin.Default()
router.Run(":8080")

// 设置哪些 IP 地址或 CIDR 被视为可信任的代理，用于设置记录真实客户端 IP 的请求头。
// 更多详情请参阅文档。
router := gin.Default()
router.SetTrustedProxies([]string{"192.168.1.2"})
```

## 基本路由规则

```go
func main() {
  // 禁用控制台颜色
  // gin.DisableConsoleColor()

  // 使用默认中间件（logger 和 recovery 中间件）创建 gin 路由
  router := gin.Default()

  router.GET("/someGet", getting)
  router.POST("/somePost", posting)
  router.PUT("/somePut", putting)
  router.DELETE("/someDelete", deleting)
  router.PATCH("/somePatch", patching)
  router.HEAD("/someHead", head)
  router.OPTIONS("/someOptions", options)

  // 默认在 8080 端口启动服务，除非定义了一个 PORT 的环境变量。
  router.Run()
  // router.Run(":3000") hardcode 端口号
}
```

### 重定向

* **HTTP 重定向（外部 / 内部 URL）**

  ```go
  c.Redirect(http.StatusMovedPermanently, "http://www.google.com/")
  ```

  * 返回 3xx 状态码
  * 浏览器重新发起请求
  * 地址栏会变化

* **POST 场景下的重定向**

  ```go
  c.Redirect(http.StatusFound, "/foo")
  ```

  * 常用 `302 Found`
  * 浏览器会转为 GET 请求

* **路由重定向（内部转发）**

  ```go
  c.Request.URL.Path = "/test2"
  router.HandleContext(c)
  ```

  * 不返回 3xx
  * 不触发新 HTTP 请求
  * 地址栏不变
  * 直接复用当前 `Context` 进入新路由

## 基本数据绑定

### 绑定 Uri
```go
package main

import "github.com/gin-gonic/gin"

type Person struct {
  ID   string `uri:"id" binding:"required,uuid"`
  Name string `uri:"name" binding:"required"`
}

func main() {
  route := gin.Default()
  route.GET("/:name/:id", func(c *gin.Context) {
    var person Person
    if err := c.ShouldBindUri(&person); err != nil {
      c.JSON(400, gin.H{"msg": err.Error()})
      return
    }
    c.JSON(200, gin.H{"name": person.Name, "uuid": person.ID})
  })
  route.Run(":8088")
}
```

### 模型绑定和验证

* **字段必须打 tag**
  绑定 JSON 用 `json:"name"`，表单用 `form:"name"`，查询参数用 `form/query`。

* **两类绑定方式**

  * **Must bind（Bind / BindJSON / BindQuery …）**
    绑定失败会：

    * 自动 `Abort`
    * 返回 **400**
    * 不可再修改响应（否则警告）
  * **Should bind（ShouldBind / ShouldBindJSON / ShouldBindQuery …）**

    * 失败只返回 `error`
    * 由开发者决定如何响应（推荐）

* **自动推断 vs 显式指定**

  * `Bind / ShouldBind`：根据 `Content-Type` 自动选择
  * `BindJSON / ShouldBindJSON`：明确指定绑定方式

* **字段校验**

  * `binding:"required"`：字段为空即报错
  * 嵌套结构体：内部字段也要加 `binding:"required"`

* **推荐实践**

  * 统一使用 `ShouldBind*`
  * 显式处理错误
  * 路由层只做绑定和校验，业务放 Service 层

```go
// 绑定 JSON
type Login struct {
  User     string `form:"user" json:"user" xml:"user"  binding:"required"`
  Password string `form:"password" json:"password" xml:"password" binding:"required"`
}

func main() {
  router := gin.Default()

  // 绑定 JSON ({"user": "manu", "password": "123"})
  router.POST("/loginJSON", func(c *gin.Context) {
    var json Login
    if err := c.ShouldBindJSON(&json); err != nil {
      c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
      return
    }

    if json.User != "manu" || json.Password != "123" {
      c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
      return
    }

    c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
  })

  // 绑定 XML (
  //  <?xml version="1.0" encoding="UTF-8"?>
  //  <root>
  //    <user>manu</user>
  //    <password>123</password>
  //  </root>)
  router.POST("/loginXML", func(c *gin.Context) {
    var xml Login
    if err := c.ShouldBindXML(&xml); err != nil {
      c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
      return
    }

    if xml.User != "manu" || xml.Password != "123" {
      c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
      return
    }

    c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
  })

  // 绑定 HTML 表单 (user=manu&password=123) (同绑定查询字符串)
  router.POST("/loginForm", func(c *gin.Context) {
    var form Login
    // 根据 Content-Type Header 推断使用哪个绑定器。
    if err := c.ShouldBind(&form); err != nil {
      c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
      return
    }

    if form.User != "manu" || form.Password != "123" {
      c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
      return
    }

    c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
  })

  // 监听并在 0.0.0.0:8080 上启动服务
  router.Run(":8080")
}
```

## 基本参数查询

下文中 `c` 指 `c *gin.Context`
* 路由参数: 通过 `c.Param("example")`查询
* 查询字符串: 通过 `c.Query("example")`  查询, 也可以用 `c.DefaultQuery("example", "default")` 定义查询默认值


