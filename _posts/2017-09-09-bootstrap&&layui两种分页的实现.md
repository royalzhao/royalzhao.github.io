---
layout: post
title: "Bootstrap&&layui两种分页的实现"
date: 2017-09-09  
description: ""
tag: 分页
---

最近做的项目中后台界面用的layui框架，前台界面用的是boostrap框架，这就导致我在做分页的过程中要考虑前台两种分页的动态实现，现在我已经爬出这个坑了，哈哈，给大家分享一下，写的不好的地方请大家批评指正！  
  
### layui框架分页  
我在做分页的过程中感觉还是layui框架的分页比较好操作的，你只要给出总条数和每页显示的数量，它将自动计算显示在页面的页数,代码如下  


  
	laypage.render({  
		elem: 'test1' //注意，这里的 test1 是 ID，不用加 # 号  
		,count: 50 //数据总数，从服务端得到  
		,limit:10 //默认是10
	});  


最终它呈现的效果如下  

![layui分页样式](/img/bootstrap&&layui_page1.png)  

看起来样式还不错吧，好了，我们言归正传，贴出我实现layui动态分页的代码  



			//首先用ajax获取后台数据总数
			$.ajax({
                url:'../diy',
                data:{
        
                },
                type:'post',
                success:function(data){
                    var sumNews = data.length;
                    $('#sumNews').html(sumNews);

                     //执行一个laypage实例
                    laypage.render({
                        elem: 'page', //注意，这里的 page 是 ID，不用加 # 号
                        count: sumNews, //数据总数 从服务端得到
                        curr:location.hash.replace('#!fenye=',''),
                        hash:'funye',
                        limit:10,
                        jump:function(obj,first){	//当我点击跳转页面时会执行另一个ajax
                            console.log(obj.curr)	//当前页数
                            console.log(obj.limit)	//每页显示条数
                            $.ajax({
                                url:'../diyfenye',
                                data:{
                                    curr:obj.curr,
                                    limit:obj.limit
                                },
                                type:'post',
                                success:function(data){
                                    //向页面渲染后台传过来的数据
                                },
                                error:function(){
                                    console.log('网络错误，通信失败');
                                }
                            })
                        }
                    });


                },
                error:function(){
                    console.log("网络错误，通信失败");
                }
            })



后台主要做的就是将前台传过来的当前页数和每页显示信息的数量获取到，使用  


	select * from 表名 limit ? , ?   


这样的sql语句来查询出所请求的数据即可，由于我后台使用notejs做的，下面贴出我后台代码，不做过多讲解，第一个ajax获取数据总数的后台比较简单，也就不贴出来了，有需要的可单独联系我获取  



	router.route('/diyfenye').post(function (req, res) {
  		console.log(req.body);
  		let sql =  `select * from diy limit ? , ? `;
		  let limit = parseInt(req.body.limit);
		  var curr = parseInt(req.body.curr);
		  if(curr <= 0){
		    curr = 1;
		  }
		  let offset = (curr-1)*limit;
		  param = [offset,limit];
		  mysql.pool.getConnection(function (error, connection) {
	    if (error) {
	      console.log({message: '连接数据库失败'})
	      return
	    }
	    connection.query({
	      sql: sql,
	      values: param
	    }, function (error, data) {
	      connection.release()
	      if (error) {
	        console.log({messsage: 'ERROR'})
	        return
	      }else{
	        res.send(data);
	      }
	      //console.log(data)
	      
	    })
	  })
	})  


这样layui的前台分页就做完了  
### bootstrap分页分享  
bootstrap分页相比于layui的分页就复杂一些了，复杂的地方在于它的页数不会自动计算，上一页下一页也需要自己来设置跳页什么的，我自己也是鼓捣了很久，下面贴出代码，由于后台程序都是一样的，不重复粘贴    
  
html部分  



	<nav>
	  	<ul class="pagination">
			
		</ul>
	</nav>  

js部分


  	
		$.ajax({
			url:'diy',	//用于获取数据总量
			data:{
	
			},
			type:'post',
			success:function(data){
				var pageSize = 2;	//每页显示数量
				var page = Math.ceil(data.length/pageSize); //数据总量/每页显示数量=页数（向上取整）  
				//console.log(page);
				var prev = '<li><a href="javascript:;" class="prev" aria-label="Previous"><span aria-hidden="true">«</span></a></li>';
				var next = '<li><a href="javascript:;" class="next" aria-label="Next"><span aria-hidden="true">»</span></a></li>'
				var pageContent='';
					
				for(var i = 1;i<=page;i++){
					pageContent += '<li><a href="FoodDIY.html?page='+i+'">'+i+'</a></li>';
					//console.log(pageContent)
				}
				$('.pagination').append(prev+pageContent+next);
				var a = location.search.split('?');
				var b = a[1].split('='); //获取地址栏page=后面的传值
				if(b[1]){
					fenyeAjax(b[1],pageSize);
				}
				$('.pagination li a').click(function(){
					if( !isNaN($(this).text())){
						fenyeAjax($(this).text(),pageSize);
					}else{
						if($(this).hasClass('prev')){
							//从地址栏获取page参数
							var a = location.search.split('?');
							var b = a[1].split('=');
							fenyeAjax(b[1]-1,pageSize);
							if((b[1]-1)<=0){
								location.href = 'FoodDIY.html?page='+1;
							}else{
								location.href = 'FoodDIY.html?page='+(b[1]-1);
							}
							
						}else{
							//从地址栏获取page参数
							var a = location.search.split('?');
							var b = a[1].split('=');
							fenyeAjax(parseInt(b[1])+1,pageSize);
							if((parseInt(b[1])+1)>page){
								location.href = 'FoodDIY.html?page='+page;
							}else{
								location.href = 'FoodDIY.html?page='+(parseInt(b[1])+1);
							}
							
						}
					}
				})
				
			},
			error:function(){
				console.log('网络错误');
			}
		})
		//分页ajax
		function fenyeAjax(curr,limit){
			$.ajax({
				url:'/diyfenye',
				data:{
					curr:curr,
					limit:limit
				},
				type:'post',
				success:function(data){
					//向页面返回查询的数据
				},
				error:function(){
					console.log('网络错误');
				}
			})
		}   

至此我的代码就呈现完毕，欢迎大家批评指正~~~