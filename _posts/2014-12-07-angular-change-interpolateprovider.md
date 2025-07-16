---
title: "Angular change interpolateProvider"
description: ""
categorys: 
tags: [Angular]
---


修改Angular的标识符，避免和Django冲突

    xxx.config(function($interpolateProvider){
      $interpolateProvider.startSymbol('{[{');
      $interpolateProvider.endSymbol('}]}');
      })
