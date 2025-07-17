---
layout: post
title: "Angular webpack-bundle-analyzer"
tagline: ""
description: ""
category: 
tags: [fontend]
---
## 起因：
编译 angular 项目时遇到文件较大的问题，之前一直没怎么查到解决方案，今日尝试优化一下。

```
chunk {0} runtime-es2015.929d39059883415e4289.js (runtime) 1.41 kB [entry] [rendered]
chunk {1} main-es2015.2f9a60d2f098ea4c9fde.js (main) 1.72 MB [initial] [rendered]
chunk {2} polyfills-es2015.e584ff88f592f04b9295.js (polyfills) 36.4 kB [initial] [rendered]
chunk {3} styles.727351c6632d6b8202b5.css (styles) 709 kB [initial] [rendered]
chunk {scripts} scripts.a234ea79a9775686d028.js (scripts) 306 kB [entry] [rendered]
Date: 2019-09-03T00:47:36.056Z - Hash: 560a590969b3968604f1 - Time: 57696ms

WARNING in budgets, maximum exceeded for initial. Budget 2 MB was exceeded by 768 kB.
                                                                                                                                               
chunk {0} runtime-es5.1164d1fc077de35adc8a.js (runtime) 1.41 kB [entry] [rendered]
chunk {1} main-es5.70f42e3efd3f730b20d4.js (main) 1.85 MB [initial] [rendered]
chunk {2} polyfills-es5.dfd2243c3f715ac10fcf.js (polyfills) 113 kB [initial] [rendered]
chunk {scripts} scripts.a234ea79a9775686d028.js (scripts) 306 kB [entry] [rendered]
Date: 2019-09-03T00:48:37.740Z - Hash: dc6083b4e00f790f40ba - Time: 61446ms

WARNING in budgets, maximum exceeded for initial. Budget 2 MB was exceeded by 268 kB.
```


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

增加编译参数 `ng build --prod --aot --vendor-chunk --common-chunk --delete-output-path --buildOptimizer --optimization`

```
chunk {0} runtime-es2015.929d39059883415e4289.js (runtime) 1.41 kB [entry] [rendered]
chunk {1} main-es2015.c94d2e3c4a649fc3a3f6.js (main) 157 kB [initial] [rendered]
chunk {2} polyfills-es2015.e584ff88f592f04b9295.js (polyfills) 36.4 kB [initial] [rendered]
chunk {3} styles.727351c6632d6b8202b5.css (styles) 709 kB [initial] [rendered]
chunk {4} vendor-es2015.d096a3f121fb11835282.js (vendor) 1.64 MB [initial] [rendered]
chunk {scripts} scripts.a234ea79a9775686d028.js (scripts) 306 kB [entry] [rendered]
Date: 2019-09-03T01:13:48.482Z - Hash: 8c4631c04b65aacfa71c - Time: 94781ms

WARNING in budgets, maximum exceeded for initial. Budget 2 MB was exceeded by 839 kB.
                                                                                                                                               
chunk {0} runtime-es5.1164d1fc077de35adc8a.js (runtime) 1.41 kB [entry] [rendered]
chunk {1} main-es5.6563023ef4a9537dac69.js (main) 161 kB [initial] [rendered]
chunk {2} polyfills-es5.dfd2243c3f715ac10fcf.js (polyfills) 113 kB [initial] [rendered]
chunk {3} vendor-es5.5d8edf1fda154ac60c2e.js (vendor) 1.77 MB [initial] [rendered]
chunk {scripts} scripts.a234ea79a9775686d028.js (scripts) 306 kB [entry] [rendered]
Date: 2019-09-03T01:15:20.251Z - Hash: 829590d7b4598a3bf62a - Time: 91169ms

WARNING in budgets, maximum exceeded for initial. Budget 2 MB was exceeded by 348 kB.
```

![参数增加后的截图](/assets/luna/ng-args-add.png)

[参数增加后的stats.json](/assets/luna/ng-args-add.json)

效果不是特别明显，总量也增加了，但是！使用了vendor，保证了在vendor不变的情况下，进行主代码的研发后，客户需要加载的总体数据将会变少，因为有缓存么。

继续找找哪里可以扣出空间的。





## 参考：

 - [vendor.bundle.js size is around 10 MB #9016](https://github.com/angular/angular-cli/issues/9016)
 - [How Did Angular CLI Budgets Save My Day And How They Can Save Yours](https://medium.com/dailyjs/how-did-angular-cli-budgets-save-my-day-and-how-they-can-save-yours-300d534aae7a)
