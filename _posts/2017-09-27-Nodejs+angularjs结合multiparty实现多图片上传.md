---
layout: post
title: "Nodejs+angularjs结合multiparty实现多图片上传"
date: 2017-09-27  
description: ""
tag: multiparty
---

这次我们说一下nodejs+angularjs多图片上传的问题  
此前也在网站看了很多篇文章，有关的内容说多不多，说少也不少，但我一一试过以后有成功的，也有没有成功的，折磨了我很长时间，最终也是成功实现了，于是想写下这篇文章，分享我的代码，也希望后人不要踏进我的坑。    

首先说一下nodejs所以依赖的插件 multiparty 和 fs，可以用npm工具来安装  
    
	npm install multiparty --save  

	npm install fs --save  

先贴出我nodejs的后台代码（注意：我的后台代码是写在路由中的）  

	const express = require('express')
    const multiparty = require('multiparty');
	const fs = require('fs');
	const router = express.Router()；
	
	router.post('/uploadImg', function (req, res,next) {
	
	  //生成multiparty对象，并配置上传目标路径
	  var form = new multiparty.Form({
	  uploadDir: './public/uploads/',//路径需要对应自己的项目更改
	  /*设置文件保存路径 */
	  encoding: 'utf-8',
	  /*编码设置 */
	  maxFilesSize: 20000 * 1024 * 1024,
	  /*设置文件最大值 20MB */
	  keepExtensions: true,
	  /*保留后缀*/
	  });
	  
	  //上传处理
	  form.parse(req, function(err, fields, files) {
	    var filesTmp = JSON.stringify(files, null, 2);
	    console.log(files);
	    
	    function isType(str) {
	      if (str.indexOf('.') == -1) {
	        return '-1';
	      } else {
	        var arr = str.split('.');
	        return arr.pop();
	      }
	    }
	    
	    if (err) {
	      console.log('parse error: ' + err);
	    } else {
	    var inputFile = files.image[0];
	    var uploadedPath = inputFile.path;
	    var type = isType(inputFile.originalFilename);
	    /*var dstPath = './public/files/' + inputFile.originalFilename;//真实文件名*/
	    var name = new Date().getTime() + '.' + type; /*以上传的时间戳命名*/
	    var dstPath = './public/uploads/' + name; /*路径需要对应自己的项目更改*/
	    console.log("type---------" + type);
	    
	    if (type == "jpg" || type == "png" || type == "exe") {
	      console.log('可以上传');
	      //重命名为真实文件名
	      fs.rename(uploadedPath, dstPath, function(err) {
	        if (err) {
	          console.log('rename error: ' + err);
	        } else {
	          console.log('上传成功');
	        }
	      });
	      res.writeHead(200, { 'content-type': 'text/plain;charset=utf-8' });
	      var data = { "code": "1",'result_code':'SUCCESS', "msg": "上传成功", "results": [{ "name": name, "path": "uploads/" + name }] };
	      console.log(JSON.stringify(data))
	      res.end(JSON.stringify(data));
	      
	      } else {
	        fs.unlink(uploadedPath, function(err) {
	        if (err) {
	        return console.error(err);
	        }
	        console.log("文件删除成功！");
	        });
	        
	        console.log('不能上传' + inputFile.originalFilename);
	        res.writeHead(200, { 'content-type': 'text/plain;charset=utf-8' });
	        var data = { "code": 0, "msg": "上传失败" };
	        res.end(JSON.stringify(data));
	      
	      }
	    }
	    
	  });
	
	});  
然后是angularjs的控制器代码  
    
	appIndex.controller('createImgs',function($rootScope,$scope,$http){
	    // 图片上传
	
		$scope.reader = new FileReader();   //创建一个FileReader接口
	    $scope.form = {     //用于绑定提交内容，图片或其他数据
	        image:{},
	    };
	    $scope.thumb = {};      //用于存放图片的base64
	    $scope.thumb_default = {    //用于循环默认的‘加号’添加图片的框
	        1111:{}
	    };
	
	    $scope.img_upload = function(files) {       //单次提交图片的函数
	        $scope.guid = (new Date()).valueOf();   //通过时间戳创建一个随机数，作为键名使用
	        $scope.reader.readAsDataURL(files[0]);  //FileReader的方法，把图片转成base64
	        $scope.reader.onload = function(ev) {
	            $scope.$apply(function(){
	                $scope.thumb[$scope.guid] = {
	                    imgSrc : ev.target.result,  //接收base64
	                }
	            });
	        };
	        
	        var data = new FormData();      //以下为像后台提交图片数据
	        data.append('image', files[0]);
			data.append('guid',$scope.guid);
			$http({
	            method: 'post',
	            url: '/uploadImg',
	            data:data,
	            headers: {'Content-Type': undefined},
	            transformRequest: angular.identity
	        }).then(function successCallBack(response) {
	            if (response.data.result_code == 'SUCCESS') {
	                $scope.form.image[$scope.guid] = response.data.results[0].path;
	                $scope.thumb[$scope.guid].status = 'SUCCESS';
	                console.log($scope.form)
	            }
	            if(data.result_code == 'FAIL'){
	                console.log(data)
	            }
	        }, function errorCallback(response) {
	            console.log('网络错误')
	        })
	    };
	
	    $scope.img_del = function(key) {    //删除，删除的时候thumb和form里面的图片数据都要删除，避免提交不必要的
	        var guidArr = [];
	        for(var p in $scope.thumb){
	            guidArr.push(p);
	        }
	        delete $scope.thumb[guidArr[key]];
	        delete $scope.form.image[guidArr[key]];
	    };
	    $scope.submit_form = function(){    //图片选择完毕后的提交，这个提交并没有提交前面的图片数据，只是提交用户操作完毕后，到底要上传哪些，通过提交键名或者链接，后台来判断最终用户的选择,整个思路也是如此
	        $http({
	            method: 'post',
	            url: '/insertImg',
	            data:$scope.form,
	        }).success(function(data) {
	            console.log(data);   
	        })
	    };
	 
	 })  
最后是html代码  

    <div class="col-md-12 col-lg-7 fill">
        <div class="form-group">
            <h4>图片</h4>
            <p>支持批量上传，支持照片的格式为<span class="label label-default">jpg & png</span> 。每张照片请不要超过<span class="label label-default">50M</span> 。为了在全屏下获得最好的效果，照片的分辨率最好大于<span class="label label-default">1920 x 1280</span> 。上传后的照片默认会按照它们的EXIF日期来排序。</p>
            <div ng-repeat="item in thumb_default">
                <!-- 这里之所以写个循环，是为了后期万一需要多个‘加号’框 -->
                <label for="one-input">
                    <div class="add">
                        <span></span>
                        <span></span>
                    </div>
                </label>
                <input type="file" id="one-input" accept="image/*" file-model="images" onchange="angular.element(this).scope().img_upload(this.files)"/>
            </div>
        </div>
        <div class="hr-text hr-text-left m-b-1 m-t-1">
            <h6 class="text-white">
                <strong>已上传</strong>
            </h6>
        </div>
        <div ng-repeat="item in thumb" class="imgload">
        <!-- 采用angular循环的方式，对存入thumb的图片进行展示 -->
            <label>
                <img ng-src="{{item.imgSrc}}"/>
            </label>
            <div class="imgDel">
                <span ng-if="item.imgSrc" ng-click="img_del($index)" class="glyphicon glyphicon-remove"></span>
            </div>
            
        </div>
       
    </div>  

添加css样式以便页面更加合理美观  

    .fill .form-group label .add{
		border:1px solid #666;
		width: 100px;
		height: 100px;
		position: relative;
		cursor: pointer;
	}
	.fill .form-group label .add span:nth-of-type(1){
		width: 2px;
		height: 50px;
		display: block;
		position: absolute;
		left: 50%;
		top: 50%;
		margin-top: -25px;
		margin-left: -1px;
		background: #666;
	}
	.fill .form-group label .add span:nth-of-type(2){
		background: #666;
		width: 50px;
		height: 2px;
		display: block;
		position: absolute;
		left: 50%;
		top: 50%;
		margin-top: -1px;
		margin-left: -25px;
	}
	.fill .form-group input{
		display: none;
	}
	.fill .imgload{
		display: inline-block;
		margin: 7px;
		position: relative;
	}
	.fill .imgload label img{
		width: 200px;
		height: 200px;
	}
	.fill .imgload .imgDel{
		width:20px;
		height: 20px;
		background: #666;
		border-radius: 50%;
		position: absolute;
		right: -10px;
		top: -10px;
		color: #ccc;
		text-align: center;
		line-height: 20px;
		cursor: pointer;
	}  
注：整体页面采用bootstrap框架布局  
欢迎大家批评指正，有问题也可以评论询问我~