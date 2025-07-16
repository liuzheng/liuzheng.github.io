---
title: "Median of Two Sorted Arrays"
description: ""
categorys: 
tags: [leetcode,python]
---


这题<https://oj.leetcode.com/problems/median-of-two-sorted-arrays/>

    C=sorted(A+B)
    l = len(C)
    if l%2==0:
        return float(C[l/2-1]+C[l/2])/2
    else:
        return C[l/2]

Runtime: 133 ms