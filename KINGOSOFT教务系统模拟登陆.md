# KINGOSOFTM教务系统模拟登陆

最近在家实在无聊,然后打开了教务系统然后对他的登陆系统分析了一下发现大概是以下步骤

![KINGOSOFT教务系统登陆](https://file.redwolf233.top/blogpic/4956a82b7f4ea92da698cc553e1e032c.png)

### 获取Cookie

首先我们可以访问教务系统的网址,然后获取Cookie的`JSESSIONID`值将其进行保存.

![Cookie](https://file.redwolf233.top/blogpic/e1c82844acf7b88561f00dec8480daa0.jpg)

### 加密账号密码信息

这时我们来分析一下网页HTML文件

```html
<td colspan=2 height="26px" align="center" valign="top">
										<input type="button" id="login" name="login" class="login" onclick="doLogon();" value=""  />
										<input type="hidden" id="reset" name="reset" class="reset" onclick="doReset();" value=""/> 
										<a href="javascript:void(0);" onclick="MobileReset();" style="padding-left: 10px;">忘记密码</a>
									</td>
```

我们可以看见当按了登陆按钮时,这是我们执行的是一个`doLogin`的JavaScript的代码



我们继续找到这个 `doLogin` 的函数,发现他在这个JavaScript文件里

```html
<script type="text/javascript" src="/suzxyjw/frame/Login.js?version=20181119"></script>
```



继续找到这个文件里的`doLogin` 函数

```js
function doLogon() {
	
	// 输入信息验证
	if (!validate()) {
		return false;
	}
	
	// 验证码正确性验证
	var username = j$("#yhmc").val();
	var password = j$("#yhmm").val();
	var randnumber = j$("#randnumber").val();
	var passwordPolicy = kutil.isPasswordPolicy(username, password);
	var url = _webRootPath + "cas/logon.action";
	
	var txt_mm_expression = document.getElementById("txt_mm_expression").value;
	var txt_mm_length = document.getElementById("txt_mm_length").value;
	var txt_mm_userzh = document.getElementById("txt_mm_userzh").value;
	
	password = hex_md5(hex_md5(password)+hex_md5(randnumber.toLowerCase()));
	/**
	var params = {
					"yhmc" : username,
					"yhmm" : password,
					"randnumber": randnumber,
					"isPasswordPolicy" : passwordPolicy
				};
	*/				
	var p_username = "_u"+randnumber;
	var p_password = "_p"+randnumber;
	username = base64encode(username+";;"+_sessionid);
	var params = p_username+"="+username+"&"+p_password+"="+password+"&randnumber="+randnumber+"&isPasswordPolicy="+passwordPolicy+
	"&txt_mm_expression="+txt_mm_expression+"&txt_mm_length="+txt_mm_length+"&txt_mm_userzh="+txt_mm_userzh ;
	//alert("params="+params);
	params = getEncParams(params); 
	//alert("encparams="+params);
	doPreLogon();					
	kutil.doAjax(url, params, doPostLogon);					
	
	function doPreLogon(){
		j$("#msg").html("正在登录......");
		j$("#login").attr("disabled", true); 
		j$("#reset").attr("disabled", true);
	}

	function doPostLogon(response) {
		var data = JSON.parse(response); 
		var status = data.status ;
		var message = data.message ;
		if ("200" == status) {
			var result = data.result ;
			window.document.location.href = result ;
		} else {
			reloadScript("kingo_encypt",_webRootPath+"custom/js/SetKingoEncypt.jsp"); 		
			if("407" == status){
				alert(message);
				showMessage("");
			}else{
				showMessage(message);
			}
			j$("#login").attr("disabled", false); 
			j$("#reset").attr("disabled", false);
			refreshImg();
			if ("401"==status) {
				j$("#randnumber").val("");
				j$("#randnumber").focus();
			} else {
				j$("#yhmc").val("");
				j$("#yhmm").val("");
				j$("#randnumber").val("");
				j$("#yhmc").focus();
			}
		}
	}
}

```

我们继续来分析

首先`username, password, randnumber` 就是你的账号密码和验证码

其次`passwordPolicy` 是一个函数的返回值我没有找到这个函数但是我用了一种方法就是在控制台来输出函数的返回值.

我随便试了一个账户和密码

![Xnip2020-04-13_19-20-29](https://file.redwolf233.top/blogpic/39ab12c545d398e499796a420461b9c3.jpg)

然后他的返回值是`0`

然后输入了正确的账号密码

![Xnip2020-04-13_19-25-18](https://file.redwolf233.top/blogpic/a2fb5404dd1a9f67ba21a6cc3dbcae52.jpg)

然后他的返回值是`1`

所以我们只需要知道`passwordPolicy`的值是`1`



接着分析 `txt_mm_expression` `txt_mm_length` ` txt_mm_userzh` 这三个变量是通过获取元素的值

```js
	var txt_mm_expression = document.getElementById("txt_mm_expression").value;
	var txt_mm_length = document.getElementById("txt_mm_length").value;
	var txt_mm_userzh = document.getElementById("txt_mm_userzh").value;
```



然后我就在之前的HTML文件中找到声明这三个变量的JavaScrpt代码

```js
	function checkpwd(oInput) {
  var pwd = oInput.value;
  var result = 0;
  for (var i = 0, len = pwd.length; i < len; ++i) {
    result |= charType(pwd.charCodeAt(i));
  }
  document.getElementById("txt_mm_expression").value = result; //密码规则
  document.getElementById("txt_mm_length").value = pwd.length; //密码长度

  var userzh = document.getElementById("yhmc").value; //取账号

  var inuserzh = "0";

  if (pwd.toLowerCase().trim().indexOf(userzh.toLowerCase().trim()) > -1) {
    inuserzh = "1";
  }
  document.getElementById("txt_mm_userzh").value = inuserzh; //判断密码是否包含账号
}

function charType(num) {
  if (num >= 48 && num <= 57) {
    return 8;
  }
  if (num >= 97 && num <= 122) {
    return 4;
  }
  if (num >= 65 && num <= 90) {
    return 2;
  }
  return 1;
}
```



通过代码分析可知`txt_mm_expression` 是一个密码的复杂度遵从以下规则

| 数字 | 大写字母 | 小写字母 | 其他符号 |
| :--: | :------: | :------: | :------: |
|  8   |    2     |    4     |    1     |

比如我的密码是以下几种情况的话, 返回值分别如下

```
123 -> 8
123A -> 10
123Aa -> 14
123Aa# -> 15
```



`txt_mm_length`就很简单了, 他就是密码的长度.



`txt_mm_userzh` 从他的注释来看就是判断密码是否包含账号.



接着我们来分析`doLogin`函数

```js
password = hex_md5(hex_md5(password)+hex_md5(randnumber.toLowerCase()));
```

`hex_md5` 函数就是把字符串变成md5值



然后他将这些变量进行`base64encode` 最后得到一个`params` 变量

```js
var p_username = "_u" + randnumber;
var p_password = "_p" + randnumber;
username = base64encode(username + ";;" + _sessionid);
var params =
  p_username +
  "=" +
  username +
  "&" +
  p_password +
  "=" +
  password +
  "&randnumber=" +
  randnumber +
  "&isPasswordPolicy=" +
  passwordPolicy +
  "&txt_mm_expression=" +
  txt_mm_expression +
  "&txt_mm_length=" +
  txt_mm_length +
  "&txt_mm_userzh=" +
  txt_mm_userzh;
```



然后将`params` 用`getEncParams` 函数加密

```js
params = getEncParams(params); 
```



然后发现他在另一个JavaScript 文件里

```html
<script type="text/javascript" src="/suzxyjw/custom/js/SetKingoEncypt.jsp?t=4901586828474712746" id="kingo_encypt"></script>
```

但是这个JavaScript 文件后面的链接t后面的参数是 刷新一次网页就会变的,所以我们要用正则表达式进行提取 `/suzxyjw/custom/js/SetKingoEncypt.jsp\?t=[0-9]+`



然后打开这个JavaScript脚本

```js
var _deskey = '6061586830242491265';
var _nowtime = '2020-04-14 10:10:42';
document.write("<script type='text/javascript' src='http://211.86.128.194:80/suzxyjw/custom/js/base64.js'></script>");
document.write("<script type='text/javascript' src='http://211.86.128.194:80/suzxyjw/custom/js/md5.js'></script>");
document.write("<script type='text/javascript' src='http://211.86.128.194:80/suzxyjw/custom/js/jkingo.des.js'></script>");
function b64_encode(data) { return base64encode(utf16to8(data)); } 
function b64_decode(data) { return utf8to16(base64decode(data)); } 
function md5(data) { return hex_md5(data); } 
function des_encode(data) { return strEnc(data, _deskey, null, null); } 
function des_decode(data) { return strDec(data, _deskey, null, null); } 
function getEncParams(params) { 
 var timestamp = _nowtime; 
 var token = md5(md5(params)+md5(timestamp)); 
 var _params = b64_encode(des_encode(params)); 
 _params = "params=" + _params + "&token="+token+"&timestamp="+timestamp; 
 return _params; 
 } 
function reloadScript00(id, jsfile){
 jsfile = jsfile + '?random='+Math.random(); 
 if (id=='kingo_encypt') document.getElementById('kingo_encypt').src = jsfile ; 
}
function reloadScript(id, jsfile){ 
 var oldS=document.getElementById(id); 
 if(oldS) oldS.parentNode.removeChild(oldS); 
 var t=document.createElement('script'); 
 if (new String(jsfile).indexOf('?') > -1){ 
 	jsfile = jsfile + '&random='+Math.random();  
 } else { 
 	jsfile = jsfile + '?random='+Math.random(); 
 }  
 t.src=jsfile; 
 t.type='text/javascript'; 
 t.id=id; 
 document.getElementsByTagName('head')[0].appendChild(t); 
}
var G_LOGIN_ID = 'kingo.guest';
```

发现`getEncParams` 这个函数里面有一个函数`des_encode` 

这个`des_encode` 里面有一个函数 `strEnc(data, _deskey, null, null)`

`strEnc` 接受四个参数 `data`, `firstKey`, `secondKey`, `thirdKey`.



> jkingo.des.js 链接我将放置文章末尾



这里有个坑就是我们访问这个`SetKingoEncypt.jsp` 链接是 我们必须要携带Cookie访问, 而且每次访问出来的`_deskey`, `_nowtime` 都是不一样的, 这里我们还是要进行字符串的提取可以用正则也可以用其他的.



### 提交登陆信息

最后通过Post 请求发送数据表单

```js
"params=" + _params + "&token=" + token + "&timestamp=" + timestamp
```

表单总共有三个数据 `params`, `token`, `timestamp`



最后发送POST 请求后可以得到如下的JSON数据

```json
{"message":"操作成功!","result":"\/suzxyjw\/MainFrm.html?random=0.14040851742858051","status":"200"}
```

最后就可以用CookIe 进行其他的操作了.



>[jkingo.des.js](http://211.86.128.194:80/suzxyjw/custom/js/jkingo.des.js)
>
>[演示代码(仅供参考)](https://github.com/52funny/KINGOSOFT-LOGIN) 

