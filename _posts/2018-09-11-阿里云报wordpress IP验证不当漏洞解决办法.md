---
layout: post
title: "阿里云报wordpress IP验证不当漏洞解决办法"
date: 2018-09-11  
description: ""
tag: wordpress
---
  



说明：
阿里云不断发短信提示网站存在漏洞，通过后台云盾想查看详细信息并修复的话还得升级到云盾专业版，我等穷人怎么买得起专业版？无奈直接网上搜索解决。

##漏洞：
###漏洞一：wordpress IP验证不当漏洞



1. 描述:wordpress /wp-includes/http.php文件中的wp_http_validate_url函数对输入IP验证不当，导致黑客可构造类似于012.10.10.10这样的畸形IP绕过验证，进行SSRF


2. 修复
找到/wp-includes/http.php这个文件，大概在文件533行，找不到可以复制下面的一部分代码，在代码编辑器里搜索一下，找到后再修改，修改文件前记得先备份http.php原文件，这是个好习惯：
    
        $same_host = strtolower( $parsed_home['host'] ) === strtolower( $parsed_url['host'] );
	    改成
	    if ( isset( $parsed_home['host'] ) ) { $same_host = ( strtolower( $parsed_home['host'] ) === strtolower( $parsed_url['host'] ) || 'localhost' === strtolower( $parsed_url['host'] ) ); } else { $same_host = false; } ;

4. 在文件的 549行左右找到

	    if ( 127 === $parts[0] || 10 === $parts[0] || 0 === $parts[0]
	    修改为：
	    if ( 127 === $parts[0] || 10 === $parts[0] || 0 === $parts[0] || 0 === $parts[0]
    
保存重新上传到服务器上就好了，已验证，可以成功。

下面的漏洞我没遇到，不过先记着，万一以后遇到了呢。

###漏洞二：wordpress后台插件更新模块任意目录遍历导致DOS漏洞

1. 描述：wordpress后台文件/wp-admin/includes/ajax-actions.php中，对代码插件路径的输入参数plugin未进行正确的规范化转义，导致黑客可传入特殊路径，造成拒绝服务。

2. 修复
找到/wp-admin/includes/ajax-actions.php。大概在文件2890附近，修改文件前记得先备份ajax-actions.php原文件

	    $plugin = urldecode( $_POST['plugin'] );
	    加上：
	    $plugin = plugin_basename( sanitize_text_field( wp_unslash( $_POST['plugin'] ) ) );
    
最后到阿里云盾控制台重新验证下漏洞

文章系转载 http://coolnull.com/4438.html

个人博客：http://www.hellozy.top   
                  [传送门](http://www.hellozy.top)