---
title: "软件测试面试题"
description: 
date: 2026-06-11T18:36:17+08:00
image: 
math: 
license: 
comments: true
draft: false
categories:
    - 软件测试
    - 面试题
tags:
    - 软件测试
    - 面试题
build:
    list: always    # Change to "never" to hide the page from the list
---

## 安全测试

### 接口防篡改如何测试

1. get 请求对params参数进行排序拼接后计算签名, 在burnsuit拦截请求后篡改params参数验证后台防篡改

2. post 请求对body(json/dataform)计算签名, 在burnsuit拦截请求后篡改body参数验证后台防篡改

### 接口重放攻击如何防止的

1. api 请求会附带时间戳, 与后台服务的时间戳+-30S差异内允许请求

2. api 请求会附带一个UUID随机数, 60S内同一个UUID请求会被后台拒绝

## 性能测试

### TPS与QPS的区别

1. TPS-每秒事务数, 一个"事务"可以包含多个步骤和系统内部调用. 例如: 云上KMS关联, 动态机密的读取, IAM用户生成, SQL代理中执行SQL等.

2. QPS-每秒查询率, 每秒对资源或list数据的单一请求次数. 例如: 图片, 用户表, 授权关系等.

## NGINX

### NG 通过 SNI 定位到是哪个服务之后, 是怎么把数据返回的

1. TCP 握手阶段 NGINX 读取到 SNI 信息, 使用对应SSL证书解密(socket A)

2. location 来确定走哪个配置

3. location 内部是 proxy_pass 情况下 ng 将自己作为客户端, 向 proxy_pass 建立新的 tcp 连接(socket B)

4. worker 进程中会将 socket A 与 socket B 建立绑定关系

5. nginx 将从 socket B 读取到的数据, 通过 SSL 证书加密后返回给用户

## TCP/IP, HTTP

### HTTP 1.1 版本中 pipelining 会有应用层队头阻塞问题, 有什么解决办法

1. 浏览器对单域名开启多个TCP连接

2. 通过 Webpack 构建工具, 将零碎的文件合并成一个单一文件, 减少请求数

3. 懒加载, 初始加载时仅加载小部分, 用户滚动页面时再发起真实的请求

### Cookie 与 Session

#### Cookie

存储于客户端, Cookie 中存储很多数据, 通过;号来间隔每个数据. 里面通常包含`JSESSIONID` `token` 个性化偏好设置, 指纹等

#### Session

存储于服务端, 可存储在内存, redis 或者 数据库中. 可存储用户身份与权限

### IPV6

1. 长度为2的128次方

2. 没有了IPV4的NAT技术

3. 浏览器访问时需要http://[IPV6]

4. 简写方式::

## Jenkins