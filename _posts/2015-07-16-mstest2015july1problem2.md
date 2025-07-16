---
title: "微软2016年校招探星夏令营第二题：最多约数问题"
description: ""
# category: 
# tags: []
---


表示对微软不感冒，然后呢也没关注，今天看到同学在玩这题，感觉很好玩，就玩玩哈。

原题：<https://hihocoder.com/contest/mstest2015july1/problem/2>

时间限制:10000ms

单点时限:1000ms

内存限制:256MB

## 描述

Given an integer n, for all integers not larger than n, find the integer with the most divisors. If there is more than one integer with the same number of divisors, print the minimum one.

## 输入
One line with an integer n.

For 30% of the data, n ≤ 103

For 100% of the data, n ≤ 1016

## 输出
One line with an integer that is the answer.

##样例输入

    100

## 样例输出

    60

常规计算机思维是将其快速解决，这里主要指人力成本上的快速，本文是在数理基础上去研究如何快速解决，这里指计算机执行效率，当然，这个的代价是人力成本的提高，不过我不赶时间。

下面做简单的建模分析：

设定 $ N = 2,3,5,7,11... (Prime Number) $, $ q_i \in N^+ $

根据约数个数计算公式，我们可以得到

$$ 
\prod_1 ^N ( q_i + 1 )
$$

则我们可以写出如下模型：

$$ 
Max : \prod_1 ^N ( q_i + 1 ) \\\\
s.t. : \prod_1 ^N ( N_i ^{q_i} ) < num 
$$

两边取对数得

$$ 
Max : \sum \log ( q_i + 1 ) \\\\
s.t. : \sum q_i \log ( N_i ) < \log ( num ) 
$$

由于$ q_i \in N^+ $，所以 $ Max: \sum \log (q_i + 1 ) \simeq Max: \sum q_i $ ， 验算发现这步不对。。。

当然，要找到符合题意的结果还需要加一个min

$$ 
Min : \sum q_i \log ( N_i ) \\\\ 
Max : \sum \log ( q_i + 1 ) \\\\
s.t. : \sum q_i \log ( N_i ) < \log ( num ) 
$$

结果么，还是不对，猿们讨论说是穷举，呵呵。这边主要还是一个非线性多目标规划问题的求解方法。

    def getPrime(prime, num):
        res = prime[-1]
        while res < num:
            res = res + 1
            p = True
            for i in prime:
                if res % i == 0:
                    p = False
                    break
            if p:
                prime.append(res)
                return prime
        return prime
    
    
    def main():
        a = input()
        prime = [2]
        result = []
        index = 0
        while True:
            if index >= 0 and len(prime) > index and a / prime[index] >= 1:
                a = a / prime[index]
                result.append(prime[index])
                index = index + 1
                if len(prime) == index:
                    prime = getPrime(prime, a)
            elif len(prime) == index or (a / prime[index] < 1 and index >= 0 and a > 1):
                index = 0
            else:
                break
        print(result)
        r = 1
        for i in result:
            r = r * i
        print(r)
    
    
    def hehe():
        num = 9097423832296800
        # num = 9127507905816300
        print(num)
        prime = [2]
        res = []
        while True:
            if num % prime[-1]==0:
                res.append(prime[-1])
                num = num / prime[-1]
                continue
            if num == 1:
                break
            prime = getPrime(prime, num)
        r = 1
        print(res)
        for i in res:
            r = r * i
        print(r)
    
    
    if __name__ == '__main__':
        # main()
        hehe()

好吧，这边帖个正确的解。

通过简单分析易得 $ q_i \geq q_{i+1} $，$ N_i = [2,3,5,7,11,13,17,19,23,29,31,37,41,43,47] $，$ 2^{60}=1152921504606846976 > 10^{16} $

所以，$ q_1 \in [0,59] , q_2 \in [0,q_1] ,\cdots ,q_N \in [0,q_{N-1}] $

然后通过更进一步分析，将取值范围进一步减小。

    prime = [2,   3,  5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]
    max   = [60, 20, 10, 6,  4,  3,  2,  2,  2,  2,  2,  2,  2,  2,  1]

由此，开始码递归！

更于2015/7/21 00:40:48

想了一晚上，还是木有想出优雅的递归。。。困。。。
