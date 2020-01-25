---
# 常用定义
title: "go gin grpc 参数自动绑定工具"           # 标题
date: 2020-01-25T18:01:00+08:00    # 创建时间
lastmod: 2020-01-25T18:01:00+08:00 # 最后修改时间
draft: false                       # 是否是草稿？
tags: ["golang","grpc", "工具", "gin"]  # 标签
categories: ["工具"]              # 分类
author: "xiexiaojun"                  # 作者

weight: 1

# 用户自定义
# 你可以选择 关闭(false) 或者 打开(true) 以下选项
comment: true   # 评论
toc: true       # 文章目录
reward: true	 # 打赏
mathjax: true    # 打开 mathjax

---

## [ginprc](https://github.com/xxjwxc/ginrpc)

[![Mentioned in Awesome Go](https://awesome.re/mentioned-badge.svg)](https://github.com/avelino/awesome-go) 

## golang gin 参数自动绑定工具
- 支持rpc自动映射
- 支持对象注册
- 支持注解路由
- 基于 [go-gin](https://github.com/gin-gonic/gin) 的 json restful 风格的golang基础库
- 自带请求参数过滤及绑定实现 binding:"required"  [validator](go-playground/validator.v8)
- 代码注册简单且支持多种注册方式
- 支持 [grpc-go](https://github.com/grpc/grpc-go) 绑定模式


## api接口说明

### 支持3种接口模式

- func(*gin.Context) //go-gin 原始接口

  func(*api.Context) //自定义的context类型

- func(*api.Context,req) //自定义的context类型,带request 请求参数

  func(*api.Context,*req)

- func(*gin.Context,*req) //go-gin context类型,带request 请求参数

  func(*gin.Context,req)

- func(*gin.Context,*req)(*resp,error) //go-gin context类型,带request 请求参数,带错误返回参数 ==> [grpc-go](https://github.com/grpc/grpc-go)

   func(*gin.Context,req)(resp,error)

## 一,参数自动绑定

```go

package main

import (
	"fmt"
	"net/http"
	"github.com/gin-gonic/gin"
	"github.com/xxjwxc/ginrpc"
	"github.com/xxjwxc/ginrpc/api"
)

type ReqTest struct {
	Access_token string `json:"access_token"`
	UserName     string `json:"user_name" binding:"required"` // 带校验方式
	Password     string `json:"password"`
}

//TestFun6 带自定义context跟已解析的req参数回调方式,err,resp 返回模式
func TestFun6(c *gin.Context, req ReqTest) (*ReqTest, error) {
	fmt.Println(req)
	//c.JSON(http.StatusOK, req)
	return &req, nil
}

func main() {
	base := ginrpc.New()
	router := gin.Default()
	router.POST("/test6", base.HandlerFunc(TestFun6))
	router.Run(":8080")
}

   ```

- curl

  ```
  curl 'http://127.0.0.1:8080/test4' -H 'Content-Type: application/json' -d '{"access_token":"111", "user_name":"222", "password":"333"}'

  ```

## 二,对象注册(注解路由)

### 初始化项目(本项目以ginweb 为名字)
	``` go mod init ginweb ```

### 代码 [详细地址>>](https://github.com/xxjwxc/ginrpc/tree/master/sample/ginweb)
```go

package main

import (
	"fmt"
	"net/http"

	_ "ginweb/routers" // debug模式需要添加[mod]/routers 注册注解路由

	"github.com/gin-gonic/gin"
	"github.com/xxjwxc/ginrpc"
	"github.com/xxjwxc/ginrpc/api"
)

type ReqTest struct {
	Access_token string `json:"access_token"`
	UserName     string `json:"user_name" binding:"required"` // 带校验方式
	Password     string `json:"password"`
}

// Hello ...
type Hello struct {
}

// Hello 带注解路由(参考beego形式)
// @router /block [post,get]
func (s *Hello) Hello(c *api.Context, req *ReqTest) {
	fmt.Println(req)
	c.JSON(http.StatusOK, "ok")
}

// Hello2 不带注解路由(参数为2默认post)
func (s *Hello) Hello2(c *gin.Context, req ReqTest) {
	fmt.Println(req)
	c.JSON(http.StatusOK, "ok")
}

// Hello3 [grpc-go](https://github.com/grpc/grpc-go) 模式
func (s *Hello) Hello3(c *gin.Context, req ReqTest) (*ReqTest, error) {
	fmt.Println(req)
	return &req,nil
}

func main() {
	base := ginrpc.New(ginrpc.WithCtx(func(c *gin.Context) interface{} {
		return api.NewCtx(c)
	}), ginrpc.WithDebug(true), ginrpc.WithGroup("xxjwxc"))

	router := gin.Default()
	base.Register(router, new(Hello))                          // 对象注册 like(go-micro)
	// or base.Register(router, new(Hello)) 
	router.Run(":8080")
}
   ```

### -注解路由相关说明

```
 // @router /block [post,get]

@router 标记  /block 路由 [post,get] method 调用方式

 ```

 #### 说明:如果对象函数中不加注解路由，系统会默认添加注解路由。post方式：带req(2个参数(ctx,req))，get方式为一个参数(ctx)



### 1. 注解路由会自动创建[mod]/routers/gen_router.go 文件 需要在调用时加：

	```
	_ "[mod]/routers" // debug模式需要添加[mod]/routers 注册注解路由

	```

	默认也会在项目根目录生成[gen_router.data]文件(保留此文件，可以不用添加上面代码嵌入)

### 2. 注解路由调用方式,支持绑定grpc函数：

	详细请看demo  [ginweb](/sample/ginweb)

### 3. 相关参数说明

	ginrpc.WithCtx ： 设置自定义context

	ginrpc.WithDebug(true) : 设置debug模式

	ginrpc.WithGroup("xxjwxc") : 添加路由前缀 (也可以使用gin.Group 分组)

	ginrpc.WithBigCamel(true) : 设置大驼峰标准(false 为web模式，_,小写)

	[更多](https://godoc.org/github.com/xxjwxc/ginrpc)

### 4. 执行curl，可以自动参数绑定。直接看结果

  ```
  curl 'http://127.0.0.1:8080/xxjwxc/block' -H 'Content-Type: application/json' -d '{"access_token":"111", "user_name":"222", "password":"333"}'
  ```

  ```
  curl 'http://127.0.0.1:8080/xxjwxc/hello.hello2' -H 'Content-Type: application/json' -d '{"access_token":"111", "user_name":"222", "password":"333"}'
  ```

## 下一步

	1.导出api文档

	2.导出postman测试配置

### 代码地址： [ginprc](https://github.com/xxjwxc/ginrpc) 如果喜欢请给星支持
