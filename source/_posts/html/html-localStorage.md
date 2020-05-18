title: 用html5的localStorage保存用户名和密码
tags: html5
categories: html5
date: 2015-07-20 11:16:56
---



## js
用法很简单 把window.localStorage 当map用就可以了。`login.account`这些都是自定义的键.

```javascript
<script type="text/javascript">
 	$("#account").change(function(){//用户名改变自动保存
		window.localStorage["login.account"]=$(this).val();
	})
	  	
  	$("#password").change(function(){//密码先判断是否有勾选保存密码才决定是否保存
  		if( $("#saveAccount").prop("checked") )
  			window.localStorage["login.pwd"]=$(this).val();
  	})
  	
 	$("#saveAccount").change(function(){
 		if( $(this).prop("checked") )//如果勾选了保存密码 保存当前密码域的数据
 			window.localStorage["login.pwd"]=$("#password").val();
 		else
 			window.localStorage["login.pwd"]="";
 	})
 	+function(){
     	if(window.localStorage["login.account"]){
     		$("#account").val(window.localStorage["login.account"]);
     	}
     	if(window.localStorage["login.pwd"]){
     		$("#password").val(window.localStorage["login.pwd"]);
     		$("#saveAccount").prop("checked",true);
     	}
 	}();
</script>
```
<!-- more -->

## html 类似的伪代码

```
 <div class="form-group">
    <label for="inputtext3" class="col-sm-2 control-label">我的账户：</label>
    <div class="col-sm-3">
    <input type="text" class="form-control" name="customer.account"  data-rule="账户:required;" 
    id="account"  autocomplete="off" placeholder="输入用户名/邮箱/手机"> 
    <span class="msg-box" for="account"></span>
    </div>
  </div>
  <div class="form-group">
    <label for="inputPassword3" class="col-sm-2 control-label">我的密码：</label>
    <div class="col-sm-3">
    <input type="password" id="password" class="form-control" data-rule="密码:required"
     name="customer.password"> 
     <span class="msg-box" for="password"></span>
    </div>
  </div>
  <div class="form-group">
    <div class="col-sm-offset-2 col-sm-3">
      <div class="checkbox">
        <label>
          <input type="checkbox" id="saveAccount"> 记住密码     
        </label>  
          <span class="pull-right"><a href="${ctx}/updatePassMail" target="_blank" class="forget-pswword ">忘记密码？</a></span>  
      </div>
  </div> 
 </div>
```