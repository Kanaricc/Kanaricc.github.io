# 康康新教务

学校换教务了?突如其来的惊喜,结果第一天教务出了个锅.

扒扒代码看看开发这个教务的公司水平,彻底失望.

这个教务系统的问题远不止时区,最大的问题是它表面上相比先前的系统...似乎...没啥...长进?

## 首页
### 编码方式
首先,来看这教务如何处理登录信息.

<!--more-->

```js
function submitForm1() {
    try {
        var xh = document.getElementById("userAccount").value;
        var pwd = document.getElementById("userPassword").value;
        if (xh == "") {
            alert("用户名不能为空！");
            return false;
        }
        if (pwd == "") {
            alert("密码不能为空！");
            return false;
        }
        var account = encodeInp(xh);
        var passwd = encodeInp(pwd);
        var encoded = account + "%%%" + passwd;
        document.getElementById("encoded").value = encoded;
        var jzmmid = document.getElementById("Form1").jzmmid;
        document.getElementById("userPassword").value = "";
        return true;
    } catch (e) {
        alert(e);
        return false;
    }
}
```

这段代码用了一种非常规的登录方式.将账号和密码编码后用`%%%`连接,作为真正的登录请求发到服务器

%%%(???)来看看这`encodeInp`是啥玩意.

```js
eval(function(p,a,c,k,e,d){e=function(c){return(c<a?"":e(parseInt(c/a)))+((c=c%a)>35?String.fromCharCode(c+29):c.toString(36))};if(!&#039;&#039;.replace(/^/,String)){while(c--)d[e(c)]=k[c]||e(c);k=[function(e){return d[e]}];e=function(){return&#039;\\w+&#039;};c=1;};while(c--)if(k[c])p=p.replace(new RegExp(&#039;\\b&#039;+e(c)+&#039;\\b&#039;,&#039;g&#039;),k[c]);return p;}(&#039;b 9="o+/=";p q(a){b e="";b 8,5,7="";b f,g,c,1="";b i=0;m{8=a.h(i++);5=a.h(i++);7=a.h(i++);f=8>>2;g=((8&3)<<4)|(5>>4);c=((5&s)<<2)|(7>>6);1=7&t;k(j(5)){c=1=l}v k(j(7)){1=l}e=e+9.d(f)+9.d(g)+9.d(c)+9.d(1);8=5=7="";f=g=c=1=""}u(i<a.n);r e}&#039;,32,32,&#039;|enc4||||chr2||chr3|chr1|keyStr|input|var|enc3|charAt|output|enc1|enc2|charCodeAt||isNaN|if|64|do|length|ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789|function|encodeInp|return|15|63|while|else&#039;.split(&#039;|&#039;),0,{}))
```

这段代码是混淆过的base64算法.**这公司开发的教务系统的登录信息居然用可逆编码,脑子呢?**


### 前端负载平衡

```js
function selectServer(uName) {
    var enableServers = true;//是否启用多服务器 true/false
    var serversArray = new Array();//服务器列表


    serversArray[0] = "http://[DELETED].10:8080/jsxsd1/";

    serversArray[1] = "http://[DELETED].11:8080/jsxsd/";

    serversArray[2] = "http://[DELETED].12:8080/jsxsd/";

    serversArray[3] = "http://[DELETED].13:8080/jsxsd/";

    serversArray[4] = "http://[DELETED].14:8080/jsxsd/";

    serversArray[5] = "http://[DELETED].15:8080/jsxsd/";

    serversArray[6] = "http://[DELETED].16:8080/jsxsd/";

    serversArray[7] = "http://[DELETED].17:8080/jsxsd/";


    var loginUrl = "xk/LoginToXk";
    if (enableServers == true) {
        if (!/[^\d]/.test(uName)) {//必须为数字
            var modVal = eval(uName % serversArray.length);
            loginUrl = serversArray[modVal] + loginUrl;
        } else {
            loginUrl = serversArray[0] + loginUrl;
        }
    } else {
        loginUrl = "/jsxsd" + loginUrl;
    }
    return loginUrl;
}
```

**教务在前端作负载平衡**

这段代码是完全面向需求编写的.

1. 需要负载只在选课时,学生学号分布均匀.
2. 大家都是正常使用系统,不会有人来搞我.

如果有任何一条不满足,就白瞎了.

不过这个`else`是什么?
```js
if (!/[^\d]/.test(uName)) {//必须为数字
    var modVal = eval(uName % serversArray.length);
    loginUrl = serversArray[modVal] + loginUrl;
} else {
    loginUrl = serversArray[0] + loginUrl;
}
```



## 后台首页
后台是一个框架嵌套的模式.另外新教务网页元素的长宽是写死的.

### 大量拼写错误

{% asset_img loadkb.png 课表请求 %}

教务的中英文混血命名法`加载课表`.

{% asset_img panle.png 拼写错误 %}

看来该公司确实适合用拼音,这个拼写错误出现了无数次.

### 后台密码加密

好吧,我承认,当我打开登录后的首页后,我确实看到了这家公司的努力.


```js
function opengld(){
    //...
	//取得userid
	var userid = &#039;[DELETED]&#039;;
	var userpsw = &#039;[DELETED]&#039;;
	//取得userid
	document.getElementById("ticket").value = userid+"#"+userpsw;
}
```

后端使用的加密方式为**没有任何加盐的MD5摘要**.

**这个是极其幼稚且严重的安全隐患**.

另外,还能看出来这套系统在开发时没有任何约定,随性写.前面登录用`%%%`,这里用`#`.

### 数据提交方式
整个后台提交数据的方式,全是隐藏表单+js赋值提交.

## 响应数据

### 时间

其HTTP服务仍然是完美的格林尼治时间,到文章发布时也没改.我不知道改下时间对于该公司运维来说难度有多大.

### Cookie

可以发现Cookie同样是MD5加密了什么玩意东西.

> 未完待续

## 新教务 旧思维

结论,**这不该是一个现代教务该有的水平**,从头到尾都透漏着不专业的表现,随便抓几个本科生多学点html和jquery和java的水平.连css和英语都不用学,毕竟这css和命名写的都是什么垃圾.

无法完全接触到后端,因此不知道后端的技术架构如何,不过看前端都写不好,大可猜测后端架构也不会怎样.单凭前端都是无论如何都是说不过去的.首先是硬伤:

* **安全意识淡薄**.这是最严重的问题.前端base64,后端md5连盐都不加,等撞库?

其他的诸如开发人员英语都没学好、命名拼音英语混合一把梭，前后端交互数据格式胡乱拼接一把梭，无移动端适配，运行维护人员不太靠谱之类的，如果说停留在系统能用就好的层面上，也不是不能接受。

其次是构建方式完全是互联网刚刚兴起时的方法。html拼接，代码vanilla一把梭，硬编码，和那个哭笑不得的负载平衡……就好像活在上个世纪。

整套系统来看,也许最大的升级是多了好几台明面上的"服务器"。

在实际选课时,新教务仍然无法承受压力,不过这有可能是带宽的锅。

如果今后发现好玩的玩意,我还会继续补充.

## 一点好玩的

不用"怀疑"了,这个公司风评不错啊看来.

[来看看知乎上的评价](https://www.zhihu.com/question/294293977)
