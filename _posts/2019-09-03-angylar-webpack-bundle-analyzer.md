---
layout: post
title: "Angular webpack-bundle-analyzer"
tagline: ""
description: ""
category: 
tags: [fontend]
---

今日编译新版 Luna ，时钟困扰我的一个编译后的包太大问题偶尔搜了一下，发现一个好玩酷帅的东西 `webpack-bundle-analyzer`。

使用教程网上自行查找，这里贴三条关键的命令

```
npm i -D webpack-bundle-analyzer
ng build --prod --stats-json
webpack-bundle-analyzer ./dist/stats-es2015.json
```

上图

![优化前截图](/assets/luna/before_analysis.png)

[优化前的stats.json](/assets/luna/before_analysis.json)







参考：

 - [How Did Angular CLI Budgets Save My Day And How They Can Save Yours](https://medium.com/dailyjs/how-did-angular-cli-budgets-save-my-day-and-how-they-can-save-yours-300d534aae7a)
