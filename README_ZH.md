# Thinkgo    [![GoDoc](https://godoc.org/github.com/tsuna/gohbase?status.png)](https://godoc.org/github.com/henrylee2cn/thinkgo)

![Lessgo Favicon](https://github.com/henrylee2cn/thinkgo/raw/master/doc/thinkgo_96x96.png)

# 概述
Thinkgo以全新的架构实现，它面向Handler接口开发，支持智能参数映射与校验、支持自动化API文档的Go语言web框架。

官方QQ群：Go-Web 编程 42730308    [![Go-Web 编程群](http://pub.idqqimg.com/wpa/images/group.png)](http://jq.qq.com/?_wv=1027&k=fzi4p1)

![thinkgo server](https://github.com/henrylee2cn/thinkgo/raw/master/doc/server.png)

![thinkgo apidoc](https://github.com/henrylee2cn/thinkgo/raw/master/doc/apidoc.png)

![thinkgo index](https://github.com/henrylee2cn/thinkgo/raw/master/doc/index.png)

# 框架下载

```sh
go get -u -v github.com/henrylee2cn/thinkgo
```

# 安装要求

Go Version ≥1.6

# 最新功能特性

- 面向Handler接口开发（func or struct），中间件与操作完全等同可任意拼接路由操作链
- 支持用struct Handler在Tag标签定义请求参数信息及其校验信息
- 由struct Handler自动构建API文档（swagger2.0）
- 支持HTTP/HTTP2、HTTPS(tls/letsencrypt)、UNIX多种Server类型
- 支持多实例运行，且配置信息相互独立
- 支持同一实例监听多Server类型、多端口
- 基于著名的httprouter构建路由器，且支持链式与树形两种路由注册风格
- 跨平台的彩色日志系统，且同时支持console和file两种输出形式（可以同时使用）
- 提供Session管理功能
- 支持Gzip全局配置
- 提供XSRF跨站请求伪造安全过滤
- 提供静态文件缓存功能
- 排版漂亮的配置文件，且自动补填默认值方便设置


# 代码示例
```
package main

import (
    "github.com/henrylee2cn/thinkgo"
    "time"
)

type Index struct {
    Id        int      `param:"in(path),required,desc(ID),range(0:10)"`
    Title     string   `param:"in(query),nonzero"`
    Paragraph []string `param:"in(query),name(p),len(1:10)" regexp:"(^[\\w]*$)"`
    Cookie    string   `param:"in(cookie),name(thinkgoID)"`
    // Picture         multipart.FileHeader `param:"in(formData),name(pic),maxmb(30)"`
}

func (i *Index) Serve(ctx *thinkgo.Context) error {
    if ctx.CookieParam("thinkgoID") == "" {
        ctx.SetCookie("thinkgoID", time.Now().String())
    }
    return ctx.JSON(200, i)
}

func main() {
  app := thinkgo.New("myapp", "0.1")

  // Register the route in a chain style
  app.GET("/index/:id", new(Index))

  // Register the route in a tree style
  // app.Route(
  //   app.NewGET("/index/:id", new(Index)),
  // )

  // Start the service
  app.Run()
}

/*
http GET:
    http://localhost:8080/index/1?title=test&p=abc&p=xyz
response:
    {
      "Id": 1,
      "Title": "test",
      "Paragraph": [
        "abc",
        "xyz"
      ],
      "Cookie": "2016-11-13 01:14:40.9038005 +0800 CST"
    }
*/
```
[All samples](https://github.com/henrylee2cn/thinkgo/raw/master/samples)

# 配置文件说明

- 应用的各实例均有单独一份配置，其文件名格式 `config/{appname}[_{version}].ini`，配置详情：

```
net_types              = normal|tls              # 多种Server类型列表，支持 normal | tls | letsencrypt | unix
addrs                  = 0.0.0.0:80|0.0.0.0:443  # 多个监听地址列表
tls_certfile           =                         # TLS证书文件路径
tls_keyfile            =                         # TLS密钥文件路径
letsencrypt_file       =                         # SSL免费证书路径
unix_filemode          = 438                     # UNIX Server的文件权限（438即0666）
read_timeout           = 0                       # 读取请求数据超时
write_timeout          = 0                       # 写入响应数据超时
multipart_maxmemory_mb = 32                      # 接收上传文件时允许使用的最大内存

[router]                                         # 路由配置区
redirect_trailing_slash   = true                 # 当前请求的URL含`/`后缀如`/foo/`且相应路由不存在时，如存在`/foo`，则自动跳转至`/foo`
redirect_fixed_path       = true                 # 自动修复URL，如`/FOO` `/..//Foo`均被跳转至`/foo`（依赖redirect_trailing_slash=true）
handle_method_not_allowed = true                 # 若开启，当前请求方法不存在时返回405，否则返回404
handle_options            = true                 # 若开启，自动应答OPTIONS类请求，可在Thinkgo中设置默认Handler

[xsrf]                                           # XSRF跨站请求伪造过滤配置区
enable = false                                   # 是否开启
key    = thinkgoxsrf                             # 加密key
expire = 3600                                    # xsrf防伪token有效时长

[session]                                        # Session配置区（详情参考beego session模块）
enable                 = false                   # 是否开启
provider               = memory                  # 数据存储方式
name                   = thinkgosessionID        # 客户端存储cookie的名字
gc_max_lifetime        = 3600                    # 触发GC的时间
provider_config        =                         # 配置信息，根据不同的引擎设置不同的配置信息
cookie_lifetime        = 0                       # 客户端存储的cookie的时间，默认值是0，即浏览器生命周期
auto_setcookie         = true                    # 是否自动设置关于session的cookie值，一般默认true
domain                 =                         # 可以访问此cookie的域名
enable_sid_in_header   = false                   # 是否将session ID写入Header
name_in_header         = Thinkgosessionid        # 将session ID写入Header时的头名称
enable_sid_in_urlquery = false                   # 是否将session ID写入url的query部分

[apidoc]                                         # API文档
enable      = true                               # 是否启用
path        = /apidoc                            # 访问的URL路径
nolimit     = false                              # 是否不限访问IP
real_ip     = false                              # 使用真实客户端的IP进行过滤
whitelist   = 192.*|202.122.246.170              # 表示允许带有`192.`前缀或等于`202.122.246.170`的IP访问
desc        =                                    # 项目描述
email       =                                    # 联系人邮箱
terms_url   =                                    # 服务条款URL
license     =                                    # 协议类型
license_url =                                    # 协议内容URL
```

- 应用只有一份全局配置，文件名为 `config/__global__.ini`，配置详情：

```
[cache]                                          # 文件内存缓存配置区
enable  = false                                  # 是否开启
size_mb = 32                                     # 允许缓存使用的最大内存（单位MB），为0时系统自动设置为512KB
expire  = 60                                     # 缓存最大时长

[gzip]                                           # gzip压缩配置区
enable         = false                           # 是否开启
min_length     = 20                              # 进行压缩的最小内容长度
compress_level = 1                               # 非文件类响应Body的压缩水平（0-9），注意文件压缩始终为最优压缩比（9）
methods        = GET                             # 允许压缩的请求方法，为空时默认为GET

[log]                                            # 日志配置区
console_enable = true                            # 是否启用控制台日志
console_level  = debug                           # 控制台日志打印水平
file_enable    = true                            # 是否启用文件日志
file_level     = debug                           # 文件日志打印水平
```

# Handler结构体字段标签说明

tag   |   key    | required |     value     |   desc
------|----------|----------|---------------|----------------------------------
param |    in    | only one |     path      | (position of param) if `required` is unsetted, auto set it. e.g. url: "http://www.abc.com/a/{path}"
param |    in    | only one |     query     | (position of param) e.g. url: "http://www.abc.com/a?b={query}"
param |    in    | only one |     formData  | (position of param) e.g. "request body: a=123&b={formData}"
param |    in    | only one |     body      | (position of param) request body can be any content
param |    in    | only one |     header    | (position of param) request header info
param |    in    | only one |     cookie    | (position of param) request cookie info, support: `http.Cookie`, `fasthttp.Cookie`, `string`, `[]byte` and so on
param |   name   |    no    |  (e.g. "id")  | specify request param`s name
param | required |    no    |   required    | request param is required
param |   desc   |    no    |  (e.g. "id")  | request param description
param |   len    |    no    | (e.g. 3:6, 3) | length range of param's value
param |   range  |    no    |  (e.g. 0:10)  | numerical range of param's value
param |  nonzero |    no    |    nonzero    | param`s value can not be zero
param |   maxmb  |    no    |   (e.g. 32)   | when request Content-Type is multipart/form-data, the max memory for body.(multi-param, whichever is greater)
regexp|          |    no    |(e.g. "^\\w+$")| param value can not be null
err   |          |    no    |(e.g. "incorrect password format")| customize the prompt for validation error

**NOTES**:
* the binding object must be a struct pointer
* the binding struct's field can not be a pointer
* `regexp` or `param` tag is only usable when `param:"type(xxx)"` is exist
* if the `param` tag is not exist, anonymous field will be parsed
* when the param's position(`in`) is `formData` and the field's type is `multipart.FileHeader`, the param receives file uploaded
* if param's position(`in`) is `cookie`, field's type must be `http.Cookie`
* param tags `in(formData)` and `in(body)` can not exist at the same time
* there should not be more than one `in(body)` param tag

# Handler结构体字段类型说明

base    |   slice    | special
--------|------------|-------------------------------------------------------
string  |  []string  | [][]byte
byte    |  []byte    | [][]uint8
uint8   |  []uint8   | multipart.FileHeader (only for `formData` param)
bool    |  []bool    | http.Cookie (only for `net/http`'s `cookie` param)
int     |  []int     | fasthttp.Cookie (only for `fasthttp`'s `cookie` param)
int8    |  []int8    | struct (struct type only for `body` param or as an anonymous field to extend params)
int16   |  []int16   |
int32   |  []int32   |
int64   |  []int64   |
uint8   |  []uint8   |
uint16  |  []uint16  |
uint32  |  []uint32  |
uint64  |  []uint64  |
float32 |  []float32 |
float64 |  []float64 |

# 扩展包
- [各种条码](https://github.com/henrylee2cn/thinkgo/raw/master/ext/barcode):       `github.com/henrylee2cn/thinkgo/ext/barcode`
- [比特单位](https://github.com/henrylee2cn/thinkgo/raw/master/ext/bitconv):       `github.com/henrylee2cn/thinkgo/ext/bitconv`
- [定时器](https://github.com/henrylee2cn/thinkgo/raw/master/ext/cron):            `github.com/henrylee2cn/thinkgo/ext/cron`
- [gorm数据库引擎](https://github.com/henrylee2cn/thinkgo/raw/master/ext/db/gorm): `github.com/henrylee2cn/thinkgo/ext/db/gorm`
- [sqlx数据库引擎](https://github.com/henrylee2cn/thinkgo/raw/master/ext/db/sqlx): `github.com/henrylee2cn/thinkgo/ext/db/sqlx`
- [xorm数据库引擎](https://github.com/henrylee2cn/thinkgo/raw/master/ext/db/xorm): `github.com/henrylee2cn/thinkgo/ext/db/xorm`
- [口令算法](https://github.com/henrylee2cn/thinkgo/raw/master/ext/otp):           `github.com/henrylee2cn/thinkgo/ext/otp`
- [UUID](https://github.com/henrylee2cn/thinkgo/raw/master/ext/uuid):              `github.com/henrylee2cn/thinkgo/ext/uuid`
- [Websocket](https://github.com/henrylee2cn/thinkgo/raw/master/ext/websocket):    `github.com/henrylee2cn/thinkgo/ext/websocket`
- [ini配置](https://github.com/henrylee2cn/thinkgo/raw/master/ini):                `github.com/henrylee2cn/thinkgo/ini`


# 开源协议
Thinkgo 项目采用商业应用友好的 [Apache2.0](https://github.com/henrylee2cn/thinkgo/raw/master/LICENSE) 协议发布。