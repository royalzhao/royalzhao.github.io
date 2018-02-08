---
layout: post
title: "computed+watch+Vuex实现多组件共享管理一个状态"
date: 2018-02-8  
description: ""
tag: Vuex
---

 好久没有写博客了，就最近写的一些东西给大家做一个分享  
要实现的就是这种效果  
![页面1](https://i.imgur.com/psgpiLR.png)  
![页面2](https://i.imgur.com/j500yqY.png)  
点击菜单栏可以弹出菜单，再点击即可关闭  

我是个菜鸟，初学vue还不太懂到底要怎么实现这个功能，通过各方查询最终决定用vuex这个状态管理模式来做，因为我的导航是一个组件，菜单栏也是一个组件，所以就涉及两个组件实时共享监听一个状态的变化  

下面贴出我的Vuex的代码  

	//store.js
    import Vue from 'vue';
	import Vuex from 'vuex';
	
	Vue.use(Vuex);
	
	const state={
	    show:false	//show的含义为菜单栏的显示与否，false为隐藏，true为显示
	}
	const mutations={  //mutations 可改变自定义show的状态
	    show(state){
	        state.show=true;
	    },
	    hide(state){
	        state.show=false;
	    }
	}
	export default new Vuex.Store({
		state,mutations
	 
	})

下面贴出导航栏的主要代码，导航栏既要改变Vuex中show的状态，也要监听show的状态，以改变导航按钮的形状  

![隐藏状态](https://i.imgur.com/mCaetVJ.png)  

![显示状态](https://i.imgur.com/3BQ9v8G.png)

	//模板部分
    <div v-if="show">
        <i class="el-icon-close" @click="dll"></i>
    </div>
    <div v-else>
        <i class="iconfont icon-caidan" @click="dll"></i>
    </div>

	//js部分
	import store from '../vuex/store'  //导入store.js,即上文提到的Vuex
    export default {
        data() {
            return {
                name: 'header',
                searchValue:'',
                face:'../../../static/img/user.jpg',
                show:false

            }
        },
        computed:{
            myValue() {
                return this.$store.state.show
            }
        },
        methods:{
            chatList(){
                this.$router.push("chatList");
            },
            blog(){
                window.location.href="http://royalzhao.github.io";
            },
            dll(){
                if(store.state.show){
                    store.commit('hide');
                    this.show = false
                }else{
                    store.commit('show');
                    this.show = true
                }
                
            },
            
        },
        store,
        watch: {
            myValue: function(newVal) {
                
                if(store.state.show){
                    this.show = true
                }else{
                    this.show = false
                }
            }
        }
    }

模板部分通过v-if判断data内部的show状态来显示对应的图标按钮  
通过computed和watch来实时监控vuex的show状态的变化，相应的改变本页面data中show的变化，从而改变图标按钮，这是导航栏改变vuex的show的状态，同时也作出对应的响应。 

下面是导航栏  

    import store from '../vuex/store'
    export default {
        data(){
            return{
                d_name:'张三',
                d_technicalTitle:'全科医生',
                d_committee:'五湖居委',
                d_tel:'17887878787',
                d_patientNum:'787',
                d_abstract:'如果你无法简洁的表达你的想法，那只说明你还不够了解它。',
                d_face:'../../../static/img/doctor.jpg'
            }
        },
        store,
        computed:{
            onRoutes(){
                return this.$route.path.replace('/','');
            },
            myValue() {
                return this.$store.state.show
            }
        },
        mounted() {
             //初始化
             this.init()
        },
        methods:{
            init(){
                var left = document.getElementById('leftXs');
                var modal = document.getElementById('modal');
                if(store.state.show){
                    left.style.marginLeft = '0';
                    modal.style.display = 'block';
                }else{
                    left.style.marginLeft = '-300px';
                    modal.style.display = 'none';
                }
            },
            talk(){
                this.$router.push('consult');
            },
            order(){
                this.$router.push('order');
            }
        },
        watch: {
            myValue: function(newVal) {
                var left = document.getElementById('leftXs');
                var modal = document.getElementById('modal');
                //其他业务代码
                if(store.state.show){
                    left.style.marginLeft = '0';
                    modal.style.display = 'block';
                }else{
                    left.style.marginLeft = '-300px';
                    modal.style.display = 'none';
                }
            },
            onRoutes:function(){
                var left = document.getElementById('leftXs');
                var modal = document.getElementById('modal');
                left.style.marginLeft = '-300px';
                modal.style.display = 'none';
                store.commit('hide');
            }
        }
        
    } 

页面先进行了初始化，判断vuex中的show状态，false为隐藏，true为显示，然后同样通过computed+watch进行监听，当show状态发生变化时做出相应的变化。  

本文重点介绍 computed+watch+Vuex实现多组件共享管理一个状态 ，对于样式以及html模板的编写则没有过多强调，需要源码进行研究的同学请移步 [https://github.com/royalzhao/sqylztc](https://github.com/royalzhao/sqylztc "社区医疗直通车")  

注：此页面为响应式网页，如需查看效果请缩小浏览器窗口  

菜鸟所编，不喜勿喷