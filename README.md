# **手机注册介绍**
> 前言

​		在Web网站中，使用手机号注册能避免一人多号的情况发生，也方便了后台对用户信息的管理，以下是介绍如何引用第三方服务到项目当中。

> 基础信息

1. 第三方依赖:

   平台使用的是阿里云平台，官方地址为：https://www.aliyun.com/?spm=5176.12825654.amxosvpfn.2.5ce12c4auR1h2m

2. 开发环境:

   Idea
   
3. 相关技术:

   JSP、Ajax、Servlet

> 基本流程

1. 后台构造随机验证码
2. 使用接口提交手机号和验证码，第三方平台发送至用户手机
3. 将第一步构造的验证码放入Session中
4. 接收用户填写的验证码，并取出第三步放入的生成验证码，两者进行对比
5. 验证码正确，请求通过，处理相关业务，否则提醒用户验证码错误

## 第三方平台引导

- 第一步:进入阿里云首页

![image-20200701220913132](https://github.com/DomModn/-/blob/master/1.png)

- 第二步:开通服务后进入控制台添加自己的签名和模板

![image-20200701221141735](https://github.com/DomModn/-/blob/master/2.png)

![image-20200701221731204](https://github.com/DomModn/-/blob/master/3.png)

​														          		   **<font color=red>注意：请严格按照规范进行申请，否则会申请失败</font>**

- 第三步:进入帮助文档

![image-20200701221848301](https://github.com/DomModn/-/blob/master/4.png)

- 第四步:创建AccessKey

![image-20200701222337213](https://github.com/DomModn/-/blob/master/5.png)

- 第五步:下载SDK（Java）

![image-20200701222519481](https://github.com/DomModn/-/blob/master/6.png)

如果是Maven项目

```xml
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-sms</artifactId>
    <version>3.0.0-rc1</version>
</dependency>
```

## 项目功能构建

前端JSP页面:

```jsp
<body>
    <form action="/dovaliCode" method="get">
        <input type="text" placeholder="请输入手机号" name="phone"/>
        <input type="button" onclick="sendCode()" id="sendCodeBtn" value="发送验证码">
        <input type="text" name="code" type="text" placeholder="请输入验证码"/>
        <input type="submit" value="注册"/>
    </form>
</body>
```

sendCode()

```javascript
			var time = 60;			//获取验证码间隔
			//获取按钮点击事件
            function sendCode() {
                //当每次time为初始值时执行
                if (time == 60) {
                    //AJax异步处理
                    sendAjax();
                }
                if (time == 0) {	//重新获取验证码
                    $("#sendCodeBtn").attr("disabled", false);
                    $("#sendCodeBtn").val("发送验证码");
                    time = 60;
                    return false;	//清除定时器
                } else {			//将获取验证码按钮不可选用
                    $("#sendCodeBtn").attr("disabled", true);
                    $("#sendCodeBtn").val(time + "秒后获取");
                    time--;
                }
                //定时器
                setTimeout(function () {
                    sendCode()
                }, 1000)
            }
```

sendAjax()

```javascript
			function sendAjax() {
                var xhr = new XMLHttpRequest();
                //声明是异步方式
                xhr.open("get", "/doGetCode?phone=" + document.getElementById("phone").value, true)
                xhr.onreadystatechange = function () {
                    //响应已就绪
                    if (xhr.readyState == 4 && xhr.readyState == 20) {
                        alert(xhr.responseText);
                    }
                }
                xhr.send();
            }
```

后端GetCodeServlet:

```java
@WebServlet("/doGetCode")   //注解方式部署Servlet
public class GetCodeServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //生成随机数
        String str = "";
        for (int i = 0; i < 4; i++) {
            str += (int) Math.floor(Math.random() * 10);
        }
        //存入生成验证码
        req.getSession().setAttribute("_code",str);
        //获取前端手机号
        String phone = req.getParameter("phone");
        //传入阿里提供接口
        sendMsg(phone,str);
    }
```

sendMsg()

```java
private void sendMsg(String phone,String str){
    	//填写基础信息
        DefaultProfile profile = DefaultProfile.getProfile("cn-hangzhou", "<accessKeyId>", "<accessSecret>");
        IAcsClient client = new DefaultAcsClient(profile);

        CommonRequest request = new CommonRequest();
        request.setSysMethod(MethodType.POST);
        request.setSysDomain("dysmsapi.aliyuncs.com");
        request.setSysVersion("2017-05-25");
        request.setSysAction("SendSms");
        request.putQueryParameter("RegionId", "cn-hangzhou");
        request.putQueryParameter("PhoneNumbers", phone);
        request.putQueryParameter("SignName", "短信签名名称");
        request.putQueryParameter("TemplateCode", "短信模板ID");
    	//短信模板变量对应的实际值，JSON格式。
        request.putQueryParameter("TemplateParam", "{'code':'"+str+"'}");
        try {
            CommonResponse response = client.getCommonResponse(request);
            System.out.println(response.getData());
        } catch (ServerException e) {
            e.printStackTrace();
        } catch (ClientException e) {
            e.printStackTrace();
        }
    }
```

ValiCodeServlet

```java
@WebServlet("/dovaliCode")
public class ValiCodeServlet extends HttpServlet{
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //code为用户输入的验证码
        String code = req.getParameter("code");
        //_code为生成的短信验证码
        String _code = (String)req.getSession().getAttribute("_code");
        //防止空指针报错
        if(_code == null){
            for (int i = 0; i < 4; i++) {
                _code += (int) Math.floor(Math.random() * 10);
            }
        }
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        String phone = req.getParameter("phone");
        resp.setContentType("text/html; charset=UTF-8");
        if(_code.equals(code)){
            //逻辑处理
        }else{
            //逻辑处理
        }
           
    }
```

## 总结

​		短信验证实现非常简单。本文档只是介绍了最基础的实现办法，实现之后，开发者自身仍然需要在此基础上进行一定的改进，比如使用验证码，服务器系统时间去限制用户短时间内获取验证码的次数，减少开发者账户发送短信的费用。


