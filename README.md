# 服务计算开发Agenda

# 笔记

## Go项目库

- [链接](https://github.com/golang/go/wiki/Projects)

## cobra

- [官方教程](https://github.com/spf13/cobra#getting-started)
- [中文教程](https://www.cnblogs.com/borey/p/5715641.html)
- 安装不成功： [clone from github](https://github.com/golang/text)
- [强制要求参数](https://github.com/spf13/cobra#required-flags)


## JSON

- [参考教程](https://blog.go-zh.org/json-and-go)

# 项目结构

项目实现**三层模型** - cmd命令层、service服务层、entity数据层；

- cmd：命令层使用cobra实现命令，具体可见命令与参数设计部分
- service: 服务层由命令层调用，处理指令对应的业务逻辑，并调用entity层的接口发出数据处理的请求 
- entity：存放数据对象以及数据处理业务
    - 用户信息: 存放在users.json, 由user.go管理，具体数据结构可见于下方 "数据结构-User信息"
    - 会议信息：存放在meetings.json, 由meeting.go管理，具体数据结构可见于下方 "数据结构-Meeting信息"
    - 当前登陆用户：存放在curUser.txt, 由curUser.go管理
    - 日志：存放在log.txt，由log.go管理


> 项目对代码进行了规范，符合Google风格的编码

> 3DES 加密方法正在调试，暂未加入

# 数据结构

## User信息

- KEY: Name string
- Password string
- Email string
- Phone string

数据结构: map, key 为 Name


## Meeting信息

- KEY: title string //会议标题
- creator string    //会议发起人
- partics []string  //会议参与者
- start_time int    //使用UNIX时间戳进行记录
- end_time int      //使用UNIX时间戳进行记录

数据结构：map, key 为 title


```
持久化要求：
使用 json 存储 User 和 Meeting 实体
当前用户信息存储在 curUser.txt 中
```
- 每次运行相关命令读取整个User/Meeting实体，并在结束时保存
- 学习使用 io/ioutils 进行文件读写
    - [文档](https://go-zh.org/pkg/io/ioutil/)
    - [教程](https://blog.csdn.net/wangshubo1989/article/details/74777112/)
- 具体的存储格式如下
    - 存储用户信息
    ```json
    {"jeff":{"Password":"dsjhjkeybdm","Email":"zys@com","Phone":"159","HostMeetings":["test1","test2"],"ParMeetings":[]},"yyh":{"Password":"123","Email":".com","Phone":"110","HostMeetings":null,"ParMeetings":["test1","test2"]}}
    ```
    - 存储会议信息     
    ```json
    {"test1":{"StartTime":1541066400,"EndTime":1541073600,"Host":"jeff","Partics":["yyh"]},"test2":{"StartTime":1541073600,"EndTime":1541080800,"Host":"jeff","Partics":["yyh"]}}
    ```


# 命令与参数设计

## help

> 显示帮助信息，系统自带

## login

> 执行登陆功能

参数列表：

- username(-u --user)
- password(-p --password)

功能：
- 若用户已登陆返回提示信息
- 从```entity/users.txt```文件中读取用户信息，确认登陆并保存状态到```curUser.txt```
- 若用户名或密码错误，返回提示信息

## register

> 执行注册功能

参数列表：
- username(-u --user)
- password(-p --password)
- phone(-ph --phone)
- email(-e --email)

功能：
- 在```entity/users.txt```中检测用户名是否重复
- 保存用户信息，自动登陆，保存登陆信息到```curUser.txt```
- 会通过向邮箱地址发送消息有无回应来检查邮箱地址的合法性
- 会通过正则表达式判断手机号的正确性
- 会判断密码的强弱并要求修改密码

## logout

> 执行登出操作

参数列表：（空）

功能：
- 检测```curUser.txt```的登陆状态，记录保存并返回提示

## queryu

> 查询已注册用户

参数列表

- username (-u --user)

功能：

- 查询已注册的用户
- 未登陆无法使用该功能

## deleteaccount

> 删除用户

参数列表：空

功能：
- 删除现在登陆的账号
- 取消所有host的会议（先删参与者，后删会议）
- 删除所有参与的会议（从会议中删除，并检测会议是否要被删除）

## createm

> 创建会议

参数列表：
- 会议标题(-t --title)
- 开始时间(-s --start)
- 结束时间(-e --end)
- 首个参与者(-p --participator)

功能：
- 以当前用户为发起人创建会议
- 检测当前用户是否登陆
- 检测会议是否存在
- 检测给定时间是否合法
- 检测开始时间和结束时间是否和发起人现有会议重叠
- 检测参与者：同adda第4、5条

## adda

> 增加会议参与者

参数列表：
- 会议标题(-t --title)
- 参与用户名(-u --user)

功能：
- 增加某个会议的参与者
- 检测当前用户是否登陆
- 检测会议是否存在
- 检测操作者是否有权限（是该会议host）
- 检测输入用户是否存在且合法（不是会议内人员 & 不是用户本身）
- 检测输入用户是否会议冲突

## removea

> 删除会议参与者

## querym

> 查询某个时段的会议

## cancelm

> 会议发起者取消某个会议

## exitm

> 会议参与者退出某个会议

参数列表：
- 会议标题(-t --title)

功能：
- 会议参与者退出会议
- 检测当前用户是否登陆
- 检测会议是否存在
- 检测用户是否host
- 检测用户是否在参与者
- 检测会议是否应被清空

## clearm

> 会议发起者清空所有发起的会议


# 项目运行截图
![](https://ws4.sinaimg.cn/large/006tNbRwgy1fwtjw5x8jcj30p00cv3yl.jpg)




