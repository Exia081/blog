---
title: CURL指南 
categories:
- LINUX命令 
tags:
- LINUX命令 
date: 2022-05-18 21:35:11
---

## 简介

大多数开发者都使用过CURL，但对它庞大的功能知之甚少，它支持的参数多达数五十多个,而且还在不断的增加

它的功能非常强大，熟练的话，完全可以满足我们日常大部分需求

## 常用参数

以下内容主要介绍它的常用参数

主要内容来自

[《curl cookbook》](https://catonmat.net/cookbooks/curl) ，
[阮一峰老师的CURL用法指南](https://www.ruanyifeng.com/blog/2019/09/curl-reference.html)
命令手册介绍，使用命令**curl --manual**就可以查看它的全部介绍

#### GET请求

GET请求相关的参数如下

| 参数             | 描述                                | 例子                                                                                                                          |
| ------------------ | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| -G               | 将-d参数的内容改为GET参数           | curl -G -d 'q=kitties' -d 'count=20' https://google.com/search<br/> 实际请求URL为https://google.com/search?q=kitties&count=20 |
| --data-urlencode | 主要用于处理GET参数中包含空格的情况 | --data-urlencode 'comment=hello world'                                                                                        |

#### POST请求

| 参数             | 描述                                            | 例子                                                                                                                                                                                                                   |
| ------------------ | ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| -d               | post 数据                                       | curl -d'login=emma＆password=123'-X POST https://google.com/login<br/>或者 curl -d 'login=emma' -d 'password=123' -X POST  https://google.com/login                                                                    |
| -d @file         | 读取文件内容作为post内容                        | curl -d '@data.txt' https://google.com/login<br/> 这样就直接读取data.txt内容向服务器发送                                                                                                                               |
| --data-urlencode | 会对post内容进行urlencode                       | curl --data-urlencode 'comment=hello world' https://google.com/login                                                                                                                                                   |
| -F               | 向服务器上传二进制文件,并且设置上传类型和文件名 | curl -F 'file=@photo.png;type=image/png;filename=me.png' https://google.com/profile<br/> 上面命令指定 MIME 类型为image/png，否则 curl 会把 MIME 类型设为application/octet-stream <br/> filename 指定服务器收到的文件名 |

#### 设置header相关参数

| 参数    | 描述                             | 例子                                                                                                                                             |
| --------- | ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| -A      | 设置user-agent                   | curl -A 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 SaHari/537.36' https://google.com |
| -H      | 指定对应的header，可以多个-H连写 | curl -H 'Accept-Language: en-US' -H 'Secret-Message: xyzzy' https://google.com                                                                   |
| -b      | 设置cookie，可以多个-b连写       | curl -b 'foo=bar' https://google.com 向服务器发送一个名为foo，值为bar的cookie                                                                    |
| -b file | 设置cookie文件                   | curl -b cookies.txt https://www.google.com                                                                                                       |
| -e      | 设置referer                      | curl -e 'https://google.com?q=example' https://www.example.com                                                                                   |
| -u      | 账号密码授权                     | curl -u 'bob:12345' https://google.com/login                                                                                                     |

#### 引导设置

| 参数 | 描述       | 例子                                                              |
| ------ | ------------ | ------------------------------------------------------------------- |
| -X   | 指定请求方式，GET，POST，HEAD，PUT等 | curl -X POST https://www.example.com                                       |
| -L   | 允许重定向，默认CURL是不跟随重定向的 | curl -L http://catonmat.net                                       |
| -x   | 设置代理   | curl -x socks5://james:cats@myproxy.com:8080 https://catonmat.net |
| -k   | 跳过HTTPS校验，不检查证书是否正确   | curl -k https://www.example.com |
| --resolve HOST:PORT:ADDRESS   | 强制解析HOST到对应的IP:PORT，非常好用，可以配合通配符使用   | curl https://datapixxxx.cn/ --resolve *:443:1.1.1.1 |
| -K 或 --config  FILE  | 指定配置文件,里面的内容按 每个参数一行进行设置   | curl -K config.txt https://www.example.com |

#### 获取内容

| 参数 | 描述       | 例子                                                              |
| ------ | ------------ | ------------------------------------------------------------------- |
| -c  file | 将服务器发送的cookie保存到对应的文件 | curl -c cookies.txt https://www.google.com                  |
| -o  file  | 将服务器响应保存到文件   |  curl -o example.html https://www.example.com |
| -v   | 打印整个请求过程，用于调试   |  curl -v https://www.example.com|
| -i   | 打印服务器响应的header   | curl -i https://www.example.com |
| -# 或者 --progress-bar   | 显示进度条   | curl -# -i https://www.example.com |

#### 调试打印

如果本地无法连接服务器，或响应很慢，我们肯定想知道是中间哪个环节出了问题

这时候就需要使用-w参数，打印一些请求过程的参数，方便我们排查问题

其中比较常用的参数如下

| 参数 | 描述       | 
| ------ | ------------ | 
| time_connect | 建立TCP连接花费的时间 |     
| time_namelookup | DNS查询花费的时间 | 
| time_total | 全部完成花费时间 | 
| remote_ip | 服务器IP | 
| remote_port | 服务器端口 | 
| time_redirect | 重定向花费时间，包括重定向的DNS查询到完成响应的时间 | 

下面是一个简单的例子

```
curl  https://www.example.com -w 'TCP连接时间:%{time_connect}\nDNS时间:%{time_namelookup}\n全部时间:%{time_total}\n服务器IP:%{remote_ip}\n'
```

响应如下：

```
TCP连接时间:1.154614
DNS时间:0.000895
全部时间:2.381030
服务器IP:93.184.216.34
```
