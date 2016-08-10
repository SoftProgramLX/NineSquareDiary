# NineSquareDiary
开发一个JavaWeb项目需要学习的技术很多，通过练习NineSquareDiary这个项目熟悉Java+JSP+MySql+Ajax+HTML5+JS+(DIV+CSS)等。开发环境是Eclipse与Tomcat v9.0<br>

###先看主要部分截图：<br>
![screen1.png](http://upload-images.jianshu.io/upload_images/301102-92a0940ab2703897.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>
![screen2.png](http://upload-images.jianshu.io/upload_images/301102-675775aa914f825f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>
![screen3.png](http://upload-images.jianshu.io/upload_images/301102-2a5e78a3c55c59a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>

###环境配置：<br>
1.Tomcat服务器配置关联项目；<br>
2.jstl下载放在lib文件夹下，包括jstl-api-1.2.jar和jstl-impl-1.2.jar。<br>
3、MySql配置。创建数据库db_9griddiary，里面包含两张表tb_diary与tb_user.对应的字段如下图。<br>
![screen4.png](http://upload-images.jianshu.io/upload_images/301102-267b1e3930a670ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>

###首页主要代码：
1.首页右边导航的JSP代码如下：
```
<div id="navigation">
	<div style="float:left;color:#006700;">
	<c:if test="${!empty sessionScope.userName}">		
		<b>   》  欢迎 ${sessionScope.userName} 登录九宫格日记网！</b>
	</c:if>
	<c:if test="${empty sessionScope.userName}">		
		<b>   》  欢迎光临九宫格日记网！</b>
	</c:if>
	</div>
	<div style="float:right;text-align: right;">
		<a href="DiaryServlet?action=listAllDiary">首页</a> 
	<c:if test="${empty sessionScope.userName}">	
		  |  <a href="#" onClick="Myopen('login')">登录</a>
		  |  <a href="#" onClick="Regopen('register')">注册</a>
		  |  <a href="forgetPwd_1.jsp">找回密码</a>	 
	</c:if>
	<c:if test="${!empty sessionScope.userName}">		
		 |  <a href="DiaryServlet?action=listMyDiary">我的日记</a>
		  |  <a href="writeDiary.jsp">写九宫格日记</a>
		  |  <a href="UserServlet?action=exit">退出登录</a>
	</c:if>
	</div>

</div>
```
2.处理登陆功能的js代码：
```
function Myopen(divID){ 								//根据传递的参数确定显示的层
     var notClickDiv=document.getElementById("notClickDiv");	//获取id为notClickDiv的层
	 notClickDiv.style.display='block';						//设置层显示
	  document.getElementById("notClickDiv").style.width=document.body.clientWidth;
	  document.getElementById("notClickDiv").style.height=document.body.clientHeight;
      document.getElementById(divID).style.display='block';							//设置由divID所指定的层显示	 
     document.getElementById(divID).style.left=(document.body.clientWidth-240)/2;		//设置由divID所指定的层的左边距
      document.getElementById(divID).style.top=(document.body.clientHeight-139)/2;		//设置由divID所指定的层的顶边框
}
function loginSubmit(form2){
	if(form2.username.value==""){		//验证用户名是否为空
		alert("请输入用户名！");form2.username.focus();return false;
	}
	if(form2.pwd.value==""){		//验证密码是否为空
		alert("请输入密码！");form2.pwd.focus();return false;
	}	
	var param="username="+form2.username.value+"&pwd="+form2.pwd.value; 	//将登录信息连接成字符串，作为发送请求的参数
	var loader=new net.AjaxRequest("UserServlet?action=login",deal_login,onerror,"POST",encodeURI(param));
}
function onerror(){
	alert("您的操作有误");
}
function deal_login(){
	/*****************显示提示信息******************************/
	var h=this.req.responseText;
	h=h.replace(/\s/g,"");	//去除字符串中的Unicode空白
	alert(h);
	if(h=="登录成功！"){
		window.location.href="DiaryServlet?action=listAllDiary";
	}else{
		form2.username.value="";//清空用户名文本框 
		form2.pwd.value="";//清空密码文本框
		form2.username.focus();//让用户名文本框获得焦点
	}
}
```

3.在UserServlet中用Servlet处理登陆请求的Java代码
```
protected void doPost(HttpServletRequest request,
                      HttpServletResponse response) throws ServletException, IOException {
    String action = request.getParameter("action");
    if ("login".equals(action)) { // 用户登录
        this.login(request, response);
    } else if ("exit".equals(action)) {// 用户退出
        this.exit(request, response);
    } else if ("save".equals(action)) { // 保存用户注册信息
        this.save(request, response);
    } else if ("getProvince".equals(action)) { // 获取省份信息
        this.getProvince(request, response);
    } else if ("getCity".equals(action)) { // 获取市县信息
        this.getCity(request, response);
    } else if ("checkUser".equals(action)) {// 检测用户名是否存在
        this.checkUser(request, response);
    } else if ("forgetPwd1".equals(action)) { // 找回密码第一步
        this.forgetPwd1(request, response);
    } else if ("forgetPwd2".equals(action)) { // 找回密码第二步
        this.forgetPwd2(request, response);
    }
}

/**
 * 功能：用户登录
 *
 * @param request
 * @param response
 * @throws ServletException
 * @throws IOException
 */
private void login(HttpServletRequest request, HttpServletResponse response)
throws ServletException, IOException {
    // TODO Auto-generated method stub
    User f = new User();
    f.setUsername(request.getParameter("username")); // 获取并设置用户名
    f.setPwd(request.getParameter("pwd")); // 获取并设置密码
    int r = userDao.login(f);
    if (r > 0) {// 当用户登录成功时
        HttpSession session = request.getSession();
        session.setAttribute("userName", f.getUsername());// 保存用户名
        session.setAttribute("uid", r);// 保存用户ID
        request.setAttribute("returnValue", "登录成功！");// 保存提示信息
        request.getRequestDispatcher("userMessage.jsp").forward(request,
                                                                response);// 重定向页面
    } else {// 当用户登录不成功时
        request.setAttribute("returnValue", "您输入的用户名或密码错误，请重新输入！");
        request.getRequestDispatcher("userMessage.jsp").forward(request,
                                                                response);// 重定向页面
    }
}
```
4.在UserDao中处理登陆功能查询数据库的java代码
```
public int login(User user) {
	int flag = 0;
	String sql = "SELECT * FROM tb_user where userName='"
			+ user.getUsername() + "'";
	ResultSet rs = conn.executeQuery(sql);// 执行SQL语句
	try {
		if (rs.next()) {
			String pwd = user.getPwd();// 获取密码
			int uid = rs.getInt(1);// 获取第一列的数据
			if (pwd.equals(rs.getString(3))) {
				flag = uid;
				rs.last(); // 定位到最后一条记录
				int rowSum = rs.getRow();// 获取记录总数
				rs.first();// 定位到第一条记录
				if (rowSum != 1) {
					flag = 0;
				}
			} else {
				flag = 0;
			}
		} else {
			flag = 0;
		}
	} catch (SQLException ex) {
		ex.printStackTrace();// 输出异常信息
		flag = 0;
	} finally {
		conn.close();// 关闭数据库连接
	}
	return flag;
}
```
5.完整代码请点击[github地址](https://github.com/SoftProgramLX/NineSquareDiary)下载<br>

QQ:2239344645    [我的github](https://github.com/SoftProgramLX?tab=repositories)<br>
