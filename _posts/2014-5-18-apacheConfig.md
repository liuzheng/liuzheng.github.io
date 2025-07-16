---
title: "Apache Config 记录"
tagline: ""
description: "Apache Config 记录"
categorys: [study]
tags: [Apache]
---


	ServerTokens OS　 `在44行 修改为：ServerTokens Prod （在出现错误页的时候不显示服务器操作系统的名称）`
	KeepAlive Off `在76行 修改为：KeepAlive On （允许程序性联机）`
	MaxKeepAliveRequests 100 `在83行 修改为：MaxKeepAliveRequests 1000 （增加同时连接数）`
	Options Indexes FollowSymLinks　 `在331行 修改为：Options Includes ExecCGI FollowSymLinks（允许服务器执行CGI及SSI，禁止列出目录）`
	DirectoryIndex index.html index.html.var `在402行 修改为：DirectoryIndex index.html index.htm Default.html Default.htm`
	Options Indexes MultiViews FollowSymLinks `在554行 修改为 Options MultiViews FollowSymLinks（不在浏览器上显示树状目录结构）`
	ServerSignature On　 `在536行 修改为：ServerSignature Off （在错误页中不显示Apache的版本）`
	#AddHandler cgi-script .cgi　`在796行 修改为：AddHandler cgi-script .cgi .pl （允许扩展名为.pl的CGI脚本运行）`
	index.php Default.php index.html.var （`设置默认首页文件，增加index.php）`
