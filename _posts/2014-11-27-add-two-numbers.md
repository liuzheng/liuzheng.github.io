---
layout: post
title: "Add Two Numbers"
description: ""
category: 
tags: []
---


    r=0
    up = 0
    while True:
        if l1==0:
            L1 = 0
        else:
            L1 = l1.val
            if l1.next == None:
                l1=0
            else:
                l1=l1.next
    
        if l2==0:
            L2 = 0
        else:
            L2 = l2.val
            if l2.next == None:
                l2=0
            else:
                l2=l2.next
        
        o=L1+L2+up
        #r.val=o%10
        up=o/10
        if r==0:
            r=ListNode(o%10)
        else:
            r.next=ListNode(o%10)
        if l1==0 and l2==0:
            if up>0:
                r.val=up
            return r

        if l1 == None: return l2
        if l2 == None: return l1
        flag = 0
        dummy = ListNode(0); p = dummy
        while l1 and l2:
            p.next = ListNode((l1.val+l2.val+flag) % 10)
            flag = (l1.val+l2.val+flag) / 10
            l1 = l1.next; l2 = l2.next; p = p.next
        if l2:
            while l2:
                p.next = ListNode((l2.val+flag) % 10)
                flag = (l2.val+flag) / 10
                l2 = l2.next; p = p.next
        if l1:
            while l1:
                p.next = ListNode((l1.val+flag) % 10)
                flag = (l1.val+flag) / 10
                l1 = l1.next; p = p.next
        if flag == 1: p.next = ListNode(1)
        return dummy.next