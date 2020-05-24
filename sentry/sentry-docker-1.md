# Sentry-Golang版 学习笔记分享
## 什么是Sentry？
```
Welcome to the Sentry documentation. Sentry is an open-source company,   
providing an application monitoring platform that helps you identify    
issues in real-time. Here we cover everything about the product, the    platform integrations, and self-hosted Sentry.    

欢迎使用Sentry文档。 Sentry是一家开源公司，提供了一个应用程序监视平台，可以帮助您实时识别问题。  
在这里，我们涵盖了有关产品，平台集成和自托管Sentry的所有内容。

【通俗讲的讲】
我们可以使用Sentry平台实时地监控我们的应用或服务、并且可以收集相关运行时错误或异常日志信息，   在第一时间将错误信息推送至我们的后台或邮件组等。这样不仅能主动帮我们第一时间发现线上问题，   而且很好的保留了异常发生时的“现场”，更有助于我们快速定位问题根源，提高解决问题的效率，    逐步提高产品的稳定性和用户体验。        
```

## 官网及文档
[https://sentry.io/welcome/](https://sentry.io/welcome/)  
[https://docs.sentry.io](https://docs.sentry.io/)  
[https://docs.sentry.io/guides/](https://docs.sentry.io/guides/)   
[https://docs.sentry.io/platforms/go/](https://docs.sentry.io/platforms/go/)    


## Sentry的原理
- 在Sentry后台注册相关账号并使用关联Dsn-key。（该key是关联应用和后台平台的桥梁）  
- 在我们的应用中潜入对应语言的SDK埋点，并关联上述key；    
  简单到仅用一个init方法就可以搞定。 
- 捕获异常并埋点，将异常信息第一时间推送至后台平台。  


## Sentry平台的优势
- 支持各种主流语言或框架。
- 跨平台性较好，并支持容器化安装搭建等。
- 平台自建成本低、部署简单、集成方便。   


## Sentry-Golang版的支持
- [官方文档](https://docs.sentry.io/platforms/go/)
- [fasthttp框架支持](https://docs.sentry.io/platforms/go/fasthttp/)   

## golang版实践笔记
### 平台搭建
- 虚拟机环境
```
CentOS Linux release 7.6.1810 (Core)
```
- Sentry Docker 镜像获取    
[Docker Official Image packaging for Sentry](https://github.com/getsentry/docker-sentry)  
[https://github.com/getsentry/onpremise](https://github.com/getsentry/onpremise)


- 安装部署  
https://github.com/docker-library/docs/tree/master/sentry  
```
1.docker run -d --name sentry-redis redis

2.docker run -d --name sentry-postgres -e POSTGRES_PASSWORD=secret -e POSTGRES_USER=sentry postgres

3.docker run --rm sentry config generate-secret-key
这一步会生成一个密钥key，比如 *l%)ti9=v#!pt__!#hpz3g33tq3hy2afpv%sda6^^ghr24)k4q ，先记下来，后面步骤中多个容器会共享该key 


4.docker run -it --rm -e SENTRY_SECRET_KEY='*l%)ti9=v#!pt__!#hpz3g33tq3hy2afpv%sda6^^ghr24)k4q' --link sentry-postgres:postgres --link sentry-redis:redis sentry upgrade


5.docker run -d -p 9090:9000 --name my-sentry -e SENTRY_SECRET_KEY='*l%)ti9=v#!pt__!#hpz3g33tq3hy2afpv%sda6^^ghr24)k4q' --link sentry-redis:redis --link sentry-postgres:postgres sentry


6.docker run -d --name sentry-cron -e SENTRY_SECRET_KEY='*l%)ti9=v#!pt__!#hpz3g33tq3hy2afpv%sda6^^ghr24)k4q' --link sentry-postgres:postgres --link sentry-redis:redis sentry run cron


7.docker run -d --name sentry-worker-1 -e SENTRY_SECRET_KEY='*l%)ti9=v#!pt__!#hpz3g33tq3hy2afpv%sda6^^ghr24)k4q' --link sentry-postgres:postgres --link sentry-redis:redis sentry run worker

检查宿主机防火墙或安全组策略，放行9090端口；

```
- 在浏览器中访问并测试
```
[http://localhost:9090/auth/login/sentry/](http://localhost:9090/auth/login/sentry/)
```

- 访问与登陆  

如果成功，您会看到如下页面      
![avatar](https://github.com/jordy1024/tool-guide/blob/master/sentry/images/access-succ.png?raw=true)  

然后用上述第4步中填入第账户登陆并配置，成功后进入首页    

![avatar](https://github.com/jordy1024/tool-guide/blob/master/sentry/images/sentry-login-index.png?raw=true)



- 简体中文设置
打开左上角第个人中心，然后点击User settings->language-> 选择简体中文即可.

- 然后可以创建一个团队，如名称为server 
![avatar](https://github.com/jordy1024/tool-guide/blob/master/sentry/images/create-team.png?raw=true)


- 然后创建一个项目，如go-sentry-test    
![avatar](https://github.com/jordy1024/tool-guide/blob/master/sentry/images/create-project2.png?raw=true)



- 将sdk潜入Golang应用，如go-sentry-test.go  
```
import (
	"errors"
	"time"
	"github.com/getsentry/sentry-go"
)

func main() {
	sentry.Init(sentry.ClientOptions{
		Dsn: "http://60e443996c464def804082d3c7e04de3@localhost:9090/2",
	})

	sentry.CaptureException(errors.New("my error"))
	// Since sentry emits events in the background we need to make sure
	// they are sent before we shut down
	sentry.Flush(time.Second * 5)
}
```
 
本地运行并模拟错误将日志上报
go run go-sentry-test.go 

- 刷新后台页面，看到刚刚咱们应用上传的error已经在issues列表中了，如图  

![avatar](https://github.com/jordy1024/tool-guide/blob/master/sentry/images/sentry-issues-upload.png?raw=true)



上述是自己搭建了sentry
如果仅是为了学习或测试一下，可以直接在官方的后台
https://sentry.io/signup/  
注册一个账号，然后将sdk潜入到自己的测试应用中看效果，如
```
package main

import (
    "log"
    "time"

    "github.com/getsentry/sentry-go"
)

func main() {
    err := sentry.Init(sentry.ClientOptions{
        // Either set your DSN here or set the SENTRY_DSN environment variable.
        Dsn: "https://xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx@xxxxxxx.ingest.sentry.io/xxxxxxx",//这里需要替换为您自己的Dsn
        // Enable printing of SDK debug messages.
        // Useful when getting started or trying to figure something out.
        Debug: true,
    })
    if err != nil {
        log.Fatalf("sentry.Init: %s", err)
    }
    // Flush buffered events before the program terminates.
    // Set the timeout to the maximum duration the program can afford to wait.
    defer sentry.Flush(2 * time.Second)
    //sentry.CaptureException("自定义运行时错误1")
    sentry.CaptureMessage("自定义error")
}
```
效果如下   
![avatar](https://github.com/jordy1024/tool-guide/blob/master/sentry/images/cloud-admin-go-test.png?raw=true)  



