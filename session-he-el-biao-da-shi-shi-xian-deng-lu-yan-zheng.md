# Session和EL表达式实现登陆验证

现在多系统的登陆都采用单点登陆了，emmmm......后期再更单点登陆的，这次由于只是个小demo，所以我们采用Session和EL表达式实现登陆验证。

#### 1.业务流程

用户在login登录页输入用户名密码-----&gt;使用用户名和密码登陆验证------&gt;验证成功，跳转欢迎页/验证失败，重新输入提示。

#### 2.数据库设计

这里就用到了一张表User表。 ![image.png](https://upload-images.jianshu.io/upload_images/9003228-bce8cc0fdff8c68c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

#### 3.项目架构（MVC）

![image.png](https://upload-images.jianshu.io/upload_images/9003228-c82d9b355e4762ba.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

#### 4.效果预览

![&#x9884;&#x89C8;.gif](https://upload-images.jianshu.io/upload_images/9003228-fab0872a25e5bb4f.gif?imageMogr2/auto-orient/strip)

#### 5.代码实现

View层: Login.jsp：

```text
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8"%>
<%
    String path = request.getContextPath();
    String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort()
            + path + "/";
%>
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="renderer" content="webkit">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
    <title>法律法规管理系统</title>
    <link rel="stylesheet" href="/public/frame/layui/css/layui.css">
    <link rel="stylesheet" href="/public/frame/static/css/style.css">

</head>

<body background="/public/frame/static/image/1.jpg">

<div class="login-main">
    <header class="layui-elip">
        <h3 style="color: bisque">法律法规管理系统</h3>
    </header>
    <form class="layui-form">
        <div class="layui-input-inline">
            <input id="username" type="text" name="username" required lay-verify="required" placeholder="账号" autocomplete="off" class="layui-input">
        </div>
        <div class="layui-input-inline">
            <input id="password" type="password" name="password" required lay-verify="required" placeholder="密码" autocomplete="off" class="layui-input">
        </div>
        <div class="layui-input-inline login-btn">
            <button type="button" class="layui-btn">登录</button>
        </div>
        <hr/>
    </form>
</div>


<script src="/public/frame/layui/layui.js"></script>
<script type="text/javascript">
    var path = '<%=basePath%>';
    layui.use(['form', 'layer'], function() {

        // 操作对象
        var form = layui.form,
            $ = layui.jquery,
            layer = layui.layer;

        // you code ...
        function dologin() {


            var username = $('input[name="username"]').val();

            if(username==null||username==""){
                layer.tips('用户名不可为空', '#username');
                return ;
            }

            var password = $('input[name="password"]').val();
            if( password==null||password==""){
                layer.tips('密码不可为空', '#password');
                return ;
            }
            if (!/[a-zA-Z0-9]{2,}/g.test(username)) {
                layer.tips('请输入正确的账号', '#username');
            } else if (!/[a-zA-Z0-9_]{4,}/g.test(password)) {
                layer.tips('请输入正确的密码', '#password');
            } else {
                var url = path+"Login/dologin";
                var data = {
                    code: username,
                    password: password
                }
                $.post(url, data, function(data) {
                    if (data.code == 1) {
                        window.location.href = path+"Index/index";
                    } else {
                        layer.msg('用户名或密码输入错误', {
                            icon: 5
                        });
                    }
                }, "json");
            }
        }

        //点击登录
        $('button[type="button"]').click(function() {
            dologin();
        });
        $('input').keydown(function(e) {
            if (e.keyCode == 13) {
                dologin();
            }
        });
    });
</script>
</body>

</html>
```

Controller层: LoginController.java：

```text
@Controller
@RequestMapping(value="Login")
public class LoginController {

    @Autowired
    private UserService userService;
    Api api=new Api();

    @RequestMapping(value="/login")
    @ResponseBody
    public ModelAndView login(){

        ModelAndView modelAndView=new ModelAndView();
        modelAndView.setViewName("/login.jsp");
        return modelAndView;
    }

    @RequestMapping(value="/dologin")
    @ResponseBody
    public Map<String,Object> dologin(HttpServletRequest request){
        String username = request.getParameter("code");
        String password = request.getParameter("password");
        if ("".equals(username)){
            return api.returnJson(2,"登陆失败,账号密码不能为空");
        }
        User user=new User();
        user.setUsername(username);
        user.setPassword(password);

        User user1=new User();
        user1=userService.findByNameAndPs(user);
        if (user1!=null) {
            HttpSession session = request.getSession();
            session.setAttribute(Const.SESSION_USERNAME, user1.getUsername());
            session.setAttribute(Const.SESSION_ID, user1.getUserId());
            session.setAttribute(Const.SESSION_USERPASSWORD, user1.getPassword());
            return api.returnJson(1,"登陆成功");
        }
        return api.returnJson(2,"登陆失败");
    }
}
```

用户点击登陆按钮，请求到dologin方法。使用账号密码查询数据库数据，看是否有匹配。

以下DAO层: UserMapper:

```text
package com.example.sl.layer.dao;

import com.example.sl.layer.model.User;
import java.util.List;

public interface UserMapper {
    int deleteByPrimaryKey(String userId);

    int insert(User record);

    User selectByPrimaryKey(String userId);

    List<User> selectAll();

    int updateByPrimaryKey(User record);

    User selectByUnameAndPs(User record);
}
```

Service层: UserService：

```text
package com.example.sl.layer.service;

import com.example.sl.layer.model.User;

public interface UserService {

    public User findByNameAndPs(User user);
}
```

UserServiceImpl:

```text
package com.example.sl.layer.service;

import com.example.sl.layer.dao.LayerMapper;
import com.example.sl.layer.dao.UserMapper;
import com.example.sl.layer.model.User;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;

@Service("userService")
@Transactional
public class UserServiceImpl  implements UserService{

    @Resource
    private UserMapper userMapper;

    @Override
    public User findByNameAndPs(User user) {
        return userMapper.selectByUnameAndPs(user);
    }
}
```

最后对应到mybatis的UserMapper.xml的sql:

```text
  <select id="selectByUnameAndPs" resultMap="BaseResultMap" parameterType="com.example.sl.layer.model.User" >
    select user_id, username, password
    from user
    where username = #{username,jdbcType=VARCHAR}
    AND password=#{password,jdbcType=VARCHAR}
  </select>
```

然后登陆的时候，我们存入了Session

```text
  HttpSession session = request.getSession();
            session.setAttribute(Const.SESSION_USERNAME, user1.getUsername());
            session.setAttribute(Const.SESSION_ID, user1.getUserId());
            session.setAttribute(Const.SESSION_USERPASSWORD, user1.getPassword());
```

这是为了方便我们在JSP中使用EL表达式，如下图: ![image.png](https://upload-images.jianshu.io/upload_images/9003228-6de8772d920f86e9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

对应的代码为:

```text
 欢迎,${sessionScope.username}
```

如果登出的话，将Session清空即可。

喜欢就给颗小💗💗吧~

