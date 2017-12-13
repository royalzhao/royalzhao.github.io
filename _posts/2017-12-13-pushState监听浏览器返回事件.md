---
layout: post
title: "pushState监听浏览器返回事件"
date: 2017-12-08  
description: ""
tag: pushState
---

## 分享一个非常好用的代码  

    <div class="modal fade bs-example-modal-sm in" id="myModal" tabindex="-1" role="dialog" aria-hidden="true" style="display: none;">
    	呀，出现了
    </div>
    <script src="https://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
    <script type="text/javascript">
    // 弹窗
    (function (window, location) {
    	
    	history.replaceState(null, document.title, location.pathname + "#!/stealingyourhistory");
    
    	history.pushState(null, document.title, location.pathname);
    
    	window.addEventListener("popstate", function () {
    
    		if (location.hash === "#!/stealingyourhistory") {
    
    			history.replaceState(null, document.title, location.pathname);
    
    			setTimeout(function () {
    				$('#myModal').css('display', 'block');
    			}, 0);
    		}
    	}, false);
    }(window, location));
    </script>  
    
当用户返回时便会出现   
> 呀，出现了  

用途广泛，大家随意去写  ：）
 


