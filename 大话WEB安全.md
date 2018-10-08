# 大话WEB安全

## 索引
* SQL脚本注入
* XSS 跨站脚本攻击
* CSRF跨站请求伪造
* HTTP劫持
* DDOS攻击
* 其他

---

## SQL脚本注入
`危险级数`☆☆☆☆☆☆   
`简介：`SQL脚本注入，就是在请求URL的参数中传入SQL语句，然后导致DAL中的语句+注入的SQL语句连接上DB进行SQL语句的执行；  
`攻击力：`轻则数据暴露，刷爆数据库，重则表数据被恶意编辑，删除，或者表被删除；   
`情景：`http://wwww.xxx.com/search?title=123 进行标题内容的查询，如果DAL层中的语句使用拼接的方式去写，
如：

``` java
//DAL
public List<MDatas> searchs(var title=""){
    if(title.isNullOrEmpty())reurn;
    var sqlStr=@"
        select fields 
        from tablename
        where title title='"+title+"'";
    //下面 链接DB执行语句返回数据table绑定对象集合 省略。。。
    //....
}
```

当我请求数据接口的时候：http://wwww.xxx.com/search?title=1' or 1=1; - -  
这个时候就会查出所有的表数据，如果我在后面插入一条删除表的语句，等。。。危险的SQL语句，那就GAME OVER了。

`防御：`
.net本身有这个安全机制，只要传入的参数是有SQL，或者JavaScript会提示危险参数，可以通过配置webconfig，或者路由方法的属性来开启和关闭。
但是最保险的方案还是要自己平时在写DAL的时候要注意，SQL语句不要使用拼接的方式，都使用参数化的方式，这样就不会出现SQL注入的问题了；
如：


``` java
//DAL
public List<MDatas> searchs(var title=""){
    if(title.isNullOrEmpty())reurn;
    var sqlStr=@"
        select fields 
        from tablename
        where title title=@title;
    //下面 链接DB执行语句返回数据table绑定对象集合 省略。。。
    //....
}

```

---

## XSS 跨站脚本攻击
`危险级数`☆☆☆☆☆☆   
`简介：`跨站的脚本攻击，就是在请求URL参数中，或者form提交的内容中注入JavaScript脚本；    
`攻击力：`轻则用户体验异常弹窗，重则用户cookie数据被盗取，引导用户到非法地址；   
场景：http://www.xx.com/userinfo/?username=吊毛&description=sb    

userinfo视图：  

``` HTML
<!-- 省略顶部 -->
<div>
    <p>@username<p>
    <p>@description</p>
</div>
<!-- 省略底部 -->
```

当我修改参数：http://www.xx.com/userinfo/?username=吊毛&description=sb&lt;script type=&quot;text/javascript&quot;&gt;alert(&#39;sb&#39;)&lt;/script&gt;

userinfo视图：  

``` HTML
<!-- 省略顶部 -->
<div>
    <p>吊毛<p>
    <p>
        sb
        <script type="text/javascript">alert('sb')</script>
    </p>
</div>
<!-- 省略底部 -->

```


这个时候把这个地址分享给别人，他一打开就会弹出一个弹窗；
如果这个时候我注入的脚本是获取cookie到我的接口，然后把地址分享给其他的用户，这样就可以通过获取到的cookie模拟用户的登陆了；
form提交就不举例了，也是一样，就是提交的内容里输入JavaScript 脚本，然后绑定内容的时候没有进行处理，这样就会导致上面一样的问题。

`防御：`在视图绑定数据的时候(前端拼接，或者服务端脚本绑定)需要对数据进行HTML编码
结果如：  
 
``` HTML
<!-- 省略顶部 -->
<div>
    <p>吊毛<p>
    <p>
        sb
        &lt;script type=&quot;text/javascript&quot;&gt;alert(&#39;sb&#39;)&lt;/script&gt;
    </p>
</div>
<!-- 省略底部 -->

```

---

## CSRF跨站请求伪造
`危险级数`☆☆☆☆☆☆   
`简介：`跨站请求伪造，就是当A站用户未退出的情况下，通过其他非法B站发起非法请求来触发A站的请求操作；用户在不知情的请求下被诱导操作    
`攻击力：`以你名义发送邮件，发消息，盗取你的账号，甚至于购买商品，虚拟货币转账，个人隐私泄露以及财产安全    
`情景：`
![此图引用hyddd博客](http://pic002.cnblogs.com/img/hyddd/200904/2009040916453171.jpg)   
具体可以看[博文](http://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html)     
`防御：`用户进入操作页面的时候绑定令牌到隐藏input，服务端进行令牌的校验，重要的操作，如：提现，充值，等，添加验证码   

--- 

## HTTP劫持
`危险级数`☆☆☆☆    
`简介：`你打开的是百度的页面，右下角弹出唐老师的不孕不育广告。        
`攻击力：`注入广告   
`情景：`在公共场所有很多免费的WIFI，有些免费的WIFI会对HTTP进行劫持，然后修改html注入广告，网络供应商也会进行HTTP劫持 ，
如使用移动网络的时候经常会出现移动广告    
`防御：`可以将HTTP替换成HTTPS这样，劫持后没有证书无法进行解密，就无法注入广告了。     


---

## DOSS攻击
`危险级数`☆☆☆☆☆☆☆     
`简介：`分布式拒绝服务攻击，俗称洪水攻击，通过木马寄生在用户机子，当成肉机，需要的时候发起群攻    
`攻击力：`刷爆服务器    
`防御：`需要再服务器部署安全防火墙。(具体方案待研究... ...)     

--- 


> 发布时间：2016-04-03