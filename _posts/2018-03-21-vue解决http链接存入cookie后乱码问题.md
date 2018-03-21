---
layout: post
title: "vue解决http链接存入cookie后乱码问题"
date: 2018-03-21  
description: ""
tag: vue
---
 在做项目的过程中，发现把一个http的链接存入cookie后，会发生乱码问题，这就导致存入数据库后也将以乱码的形式存在。我的问题如下   

将  `http://127.0.0.1:4000/upload/1521527833806.jpg`  这样一个图片链接存入到cookie中  
发现存入后变成了  `http%3A//127.0.0.1%3A4000/upload/1521527833806.jpg` 这样   
冒号的部分变成了 `%3A  ` 
因为后面我还要把这个链接存入到数据库，并且还要调用  
结果就导致调用出来的图片链接根本无法显示   
之前想到的办法是将cookie中的图片链接进行解码变成：（因为我知道cookie只是不识别：，把它变成了unicode），然而操作了半天也没有成功，最终没有再管它，而是在从数据库里调用图片的时候进行替换，把链接中所有 `%3A` 的部分都变成`：`

下面贴出代码  

    //模板部分
    <img :src="item.face | replace" alt="">
    	
    //js部分  与data()属性并列
    filters:{
	    replace (input){
	    	return input.replace(/%3A/g, ':')
	    }
       
    }  

做个笔记，备忘