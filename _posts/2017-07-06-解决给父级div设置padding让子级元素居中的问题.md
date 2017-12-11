---
layout: post
title: "解决给父级div设置padding让子级元素居中的问题"
date: 2017-07-06  
description: ""
tag: padding
---

在做网页过程中，我们常常会遇到让子级块状div居中的问题，如下图  
![](/img/css3_padding1.png)  
代码  

    <!doctype html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>无标题文档</title>
    <style type="text/css">
    .box{
    	margin: 20px 20px;
    	width: 200px;
    	height: 200px;
    	background: #00A72B;
    	border: 5px solid #007FFF;
    
    }
    .box .box-content{
    	width: 50px;
    	height: 50px;
    	background: #3399FF;
    }
    </style>
    </head>
    
    <body>
    	<div class="box">
    		<div class="box-content"></div>
    	</div>
    
    </body>
    </html>  
    
我们想让这个蓝色的小块水平垂直居中，不考虑其它方法，只用padding的方式，设置padding值，让它可以水平垂直居中，发现无论怎么设置padding的值都无法让它居中，且父级div还会越来越大，如图  
![](/img/css3_padding2.png)  
那么重点来了，只要设置box-sizing: border-box;就可以解决父级div不断变大的问题，css样式如下  

    <style type="text/css">
    .box{
    	margin: 20px 20px;
    	width: 200px;
    	height: 200px;
    	background: #00A72B;
    	border: 5px solid #007FFF;
    	padding: 50px;
    	box-sizing: border-box;
    }
    .box .box-content{
    	width: 50px;
    	height: 50px;
    	background: #3399FF;
    }
    </style>  
    
结果如图  
![](/img/css3_padding3.png)  
下面只要我们设置padding为70px,就可以让它垂直居中啦，是不是很给力啊！  
![](/img/css3_padding4.png)
