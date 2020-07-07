# regin框架 #
regin是一款基于go-gin框架封装的web框架,用于快速构建web应用和后端服务.
### 目录
- [安装与配置](https://github.com/go-touch/regin)  
- 快速开始
- 项目结构
- 路由配置
- 服务配置
- Web应用

### 安装与配置  
#### 1. 安装Go (version 1.10+), 然后可使用下面命令进行安装regin
	$ go get github.com/go-touch/regin
#### 2. 设置环境变量      
	REGIN_RUNMODE = dev | test | prod
#### 3. 如使用go mod包依赖管理工具,请参考下面命令
##### Windows 下开启 GO111MODULE 并设置 GOPROXY 的命令为：
	$ set GO111MODULE = on
	$ go env -w GOPROXY = https://goproxy.cn,direct
##### MacOS 或者 Linux 下开启 GO111MODULE 并设置 GOPROXY 的命令为：
	$ export GO111MODULE = on
	$ export GOPROXY = https://goproxy.cn
### 快速开始
####入口文件(xxx 代表项目名称,后面也是)
	$ cat ./xxx/main.go

	package main
	
	import (
		"github.com/go-touch/regin"
		_ "xxx/application/router"
	)
	
	func main() {
		regin.Guide.HttpService()
	}

### 项目结构
- application
    - modules // action存放目录,
  	    - demo // 模块名
    - model
        - bussiness // 业务类
        - common // 公共处理函数
        - dao // 抽象数据模型
        - form // 校验数据模型
        - mysql // mysql表字段映射模型
        - redis // redis数据模型
    - router // 注册路由, 主要通过map存储modules对应的action实例的映射
- config
    - dev // 必要配置项(通过系统环境变量选择配置路径)
        - database.ini
        - server.ini
        - redis.ini
    - test
    - prod
- runtime
    - log // 存储系统错误日志
- main.go // 入口文件

### 路由配置
#### 举例: xxx/application/router/demo.go 
	package router

	import (
		"github.com/go-touch/regin/base"
		"xxx/application/modules/demo"
		"xxx/application/modules/api/role"
	)
	
	func init() {
		base.Router.General("demo", base.GeneralMap{
			"v1.index":     &demo.Index{}, // 对应 modules下的index结构体
			"role.checkapi": &role.CheckApi{}, // 对应 modules下的checkapi结构体
		})
	}

#### 路径访问
	格式： http://127.0.0.1/module/controller/action
	例如: http://127.0.0.1/demo/v1/index
	备注：regin中的路由比较松散,url中pathinfo采用三段路径, 通过取三段信息,使用 . 拼接作为key, 读取路由map里面对应的action, 
	因此路径的含义可依据路由配置定义,并无严格规定.
### 服务配置
#### xxx/config/server.ini
	; 主配置
	[main]
	httpHost = 127.0.0.1:8080 // http服务地址端口
	httpsHost = 127.0.0.1:443 // https服务地址端口
	
	; 错误配置
	[error]
	log = true // 是否开启错误日志
	读取format = {date}-Error // 日志文件名格式
	pattern = local // 日志存储方式 local:本地 remote:远程
#### 项目中使用配置
	读取方式： (不会直接获取对应值,而是返回一个regin的configValue结构体指针，可实现对应类型转换)
	config ：= service.App.GetConfig("xxx.xxx.xxx") 

	调用示例: 
	config := service.App.GetConfig("server.main.httpHost").ToString()

	备注: server.ini、database.ini、redis.ini等必要配置项,字段均为框架使用,不可更改.
		
### Web应用
#### 基类action的代码示例:
	package base
	
	type AppAction interface {
		BeforeExec(request *Request) (result *Result)
		Exec(request *Request) (result *Result)
	}
	
	type Action struct {
		AppAction
	}
	
	// Before action method. // 该方法会在调用方法 Exec 前执行, 可通过其实现token验证、鉴权等业务,通过返回值*result,控制响应结果
	func (a *Action) BeforeExec(request *Request) (result *Result) { 
		return
	}
	
	// Action method. // 该方法为action主业务逻辑方法,用于实现具体的业务
	func (a *Action) Exec(request *Request) (result *Result) { 
		return
	}
#### 项目action的代码示例:	
	$ cat xxx/application/modules/demo/mysql_select
	package demo
	
	import (
		"github.com/go-touch/regin/app/db"
		"github.com/go-touch/regin/base"
	)
	
	type MysqlSelect struct {
		base.Action
	}
	
	// 执行方法
	func (this *MysqlSelect) Exec(request *base.Request) *base.Result {
		result := base.JsonResult()
	
		// 查询一条数据
		ret := db.Model("PlusArticle").FetchRow(func(dao *db.Dao) {
			dao.Where("id", 202)
		})
		
		// 错误处理
		if err := ret.ToError(); err != nil {
			result.SetData("code", 1000)
			result.SetData("msg", "系统出了点问题~")
			return result
		}
	
		// json输出数据库查询结果
		result.SetData("data", ret.ToStringMap())
		return result
	}
