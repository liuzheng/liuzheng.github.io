---
layout: post
title: "MATLAB PageRank小结"
tagline: ""
description: "MATLAB PageRank"
category: study
tags: [MATLAB, PageRank]
---


从网上找到 [Page-Rank-Matlab-Code](http://people.revoledu.com/kardi/tutorial/PageRank/Page-Rank-Matlab-Code.html) 提及 MATLAB 的 PageRank 代码，使用 `profile viewer` 发现有一段 `for` 效率极低

源代码如下

    function p=PageRank(L,d) 
    % return PageRank vector 
    % 
    % input: 
    % L = Link Matrix (=rotated adjacency matrix) 
    % d = constant parameter 
    % 
    % Example from Kardi Teknomo's Page Rank tutorial 
    % is given as input and output of this function. 
    % Read the full tutorial for more explanation. 
    % 
    % (c) 2012 Kardi Teknomo 
    % http://people.revoledu.com/kardi/tutorial/ 
    %%%%%%%%%%%%%%%%%%%%%%% 
    if nargin<1 
        L=[1 1 1 1 1 1; 
           0 1 1 1 1 1; 
           0 0 1 1 1 1; 
           0 0 0 1 1 1; 
           0 0 0 0 1 1; 
           0 0 0 0 0 1;]; 
    end 
    if nargin<2, 
        d=0.85; 
    end 
      
    [m,n]=size(L); 
    c=sum(L); 
    L_c=L./repmat(c,m,1); 
    k=0; 
      
    while 1 
        k=k+1; 
        for i=1:m
            p(i)=(1-d)+d*(L_c(i,:)*c'); 
        end
        c=p; 
        if sum(sum(p))==m || k>256, 
            break; 
        end
    end
	
修改如下

    function p=PageRank(L,d) 
    % return PageRank vector 
    % 
    % input: 
    % L = Link Matrix (=rotated adjacency matrix) 
    % d = constant parameter 
    % 
    % Example from Kardi Teknomo's Page Rank tutorial 
    % is given as input and output of this function. 
    % Read the full tutorial for more explanation. 
    % 
    % (c) 2012 Kardi Teknomo 
    % http://people.revoledu.com/kardi/tutorial/ 
    %%%%%%%%%%%%%%%%%%%%%%% 
    if nargin<1 
        L=[1 1 1 1 1 1; 
           0 1 1 1 1 1; 
           0 0 1 1 1 1; 
           0 0 0 1 1 1; 
           0 0 0 0 1 1; 
           0 0 0 0 0 1;]; 
    end 
    if nargin<2, 
        d=0.85; 
    end 
      
    [m,n]=size(L); 
    c=sum(L); 
    L_c=L./repmat(c,m,1); 
    k=0; 
      
    while 1 
        k=k+1; 
        p=(1-d)+(L_c(:,:)*c')*d;
        c=p'; 
        if sum(sum(p))==m || k>256, 
            break; 
        end
    end