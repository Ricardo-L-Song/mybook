# ThinkPHP5实现答题管理系统（一）

最近实习做了一个PHP的项目练手，大概是一个答题管理的模板\(已部署至www.songlei.online\)，用了TP5+Jquery+layui来实现，由于整个系统功能模块有点多，所以我们逐个拆分出来。这次先做问题模板的添加和删除。如下图。

![&#x6A21;&#x677F;.JPG](https://upload-images.jianshu.io/upload_images/9003228-6552a5e1126d1082.JPG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

### 1.前期准备

TP5框架：在tp5的官网下载框架模板[ThinkPHP5核心版](http://www.thinkphp.cn/down/1156.html) 编辑器:我选用的是VS code。因为相较于其它IDE，更加轻量级,别的集成IDE亦可。 一点点的TP5知识储备:[TP5完全开发手册](https://www.kancloud.cn/manual/thinkphp5/118003) 一点点的layui文档的bang助:[layui开发使用文档](http://www.layui.com/doc/)

### 2.架构设计\(MVC\)

![image.png](https://upload-images.jianshu.io/upload_images/9003228-4291c29959d2dfaf.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

这里我选择单模块index模块 将所有控制器都放在Controller层 不同于TP5框架提供的index模块的view层，我将view层要渲染的html页面都放在了template目录下，并且与model层一一对应，配置的代码，在application模块下的config.php中。

```text
  'template'               => [
        // 模板引擎类型 支持 php think 支持扩展
        'type'         => 'Think',
        // 模板路径
        'view_path'    => '',
        // 模板后缀
        'view_suffix'  => 'html',
        // 模板文件名分隔符
        'view_depr'    => DS,
        // 模板引擎普通标签开始标记
        'tpl_begin'    => '{',
        // 模板引擎普通标签结束标记
        'tpl_end'      => '}',
        // 标签库标签开始标记
        'taglib_begin' => '{',
        // 标签库标签结束标记
        'taglib_end'   => '}',
          'view_base'    => ROOT_PATH . 'template/',
    ],
```

在template关联数组末尾加一句view\_base的关联

```text
    'view_base'    => ROOT_PATH . 'template/',
```

### 3.数据库设计

![image.png](https://upload-images.jianshu.io/upload_images/9003228-8553db1639c7c3d5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

t_gr_是表前缀，总共有四张表。 qsn\_model表用来存储题目的模板数据 qsn用来存储每套模板下的题目数据 qsn\_detail用来存储用来存储每个题目对应选型的数据 psg\_qsn\_r用来存储用户答题的数据

今天的主角是qsn\_model表 ![image.png](https://upload-images.jianshu.io/upload_images/9003228-9fe6df7433a8b092.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9003228-3693faba834daead.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

四个字段，主键\(model\_id\)，模板名称\(name\)，模板创建时间\(time\)，创建人名称\(create\_name\)。

### 4.View层实现（Jquery+layui）

首先是添加模板的View层实现。

![image.png](https://upload-images.jianshu.io/upload_images/9003228-a9ef7ffc3cd73220.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

引用了layui的按钮组样式 id为btn-add的按钮 即为添加模板按钮 ![image.png](https://upload-images.jianshu.io/upload_images/9003228-f975b65118b15b6f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

点击添加模板 我们用Jquery设置其弹出了一个layui的弹出层 id为set-add-put

```text
            //弹出添加窗口
            $('#btn-add').click(function() {
                layer.open({
                    type: 1,
                    skin: 'layui-layer-rim', //加上边框
                    area: ['660px', '350px'], //宽高
                    content: $('#set-add-put'),
                    title: "添加模板"
                });
            });
```

如下

![image.png](https://upload-images.jianshu.io/upload_images/9003228-2f5558315f91fb44.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

input框有三个，分别对应数据库的create\_name,time,name。

```text
 <!--添加弹出层-->
    <div id="set-add-put" style="display:none; width:550px; padding:20px;">
        <form class="layui-form">
            <div class="layui-form-item">
                <label class="layui-form-label">创建人名字</label>
                <div class="layui-input-block">
                    <input type="text" name="create_name" required lay-verify="required" placeholder="请输入创建人姓名" autocomplete="off" class="layui-input">
                </div>
            </div>
            <div class="layui-form-item">
                <label class="layui-form-label">创建时间</label>
                <div class="layui-input-block">
                    <input type="text" name="time" required lay-verify="required" placeholder="请输入创建时间" autocomplete="off" class="layui-input" id="time">
                </div>
            </div>
            <div class="layui-form-item">
                <label class="layui-form-label">模板名称</label>
                <div class="layui-input-block">
                    <input type="text" name="name" required lay-verify="required" placeholder="请输入模板名称" autocomplete="off" class="layui-input">
                </div>
            </div>
            <div class="layui-form-item">
                <div class="layui-input-block">
                    <button type="button" class="layui-btn" lay-submit lay-filter="formDemo" id="add">立即添加</button>
                    <button type="reset" class="layui-btn layui-btn-primary">重置</button>
                </div>
            </div>
        </form>
    </div>
```

然后，我们数据点击立即添加按钮，id为add。我们对其用Jquery进行ajax请求。

```text
 //添加数据
            $('#add').click(function() {
                var create_name = $('input[name="create_name"]').val(); //获取值
                var name = $('input[name="name"]').val();
                var time = $('input[name="time"]').val();
                if (create_name !== '') {
                    //打开正在加载中弹出层
                    layer.msg('加载中', {
                        icon: 16,
                        shade: 0.01,
                        time: '9999999'
                    });
                    var url = "{:url('survey/add_qsn')}";
                    var data = {
                        create_name: create_name,
                        name: name,
                        time: time
                    }
                    $.post(url, data, function(data) { //使用ajax提交
                        layer.closeAll();
                        if (data.code == 1) { //这里的code对应返回的状态码
                            layer.msg(data.msg, {
                                icon: 6
                            });
                            location.reload();
                        } else {
                            layer.msg(data.msg, {
                                icon: 5
                            });
                        }
                    }, "json");
                }
            });
```

提交的data，就是我们输入框获取的三个值，create\_name,name,time。 提交到Controller层，如果返回的数据状态码为代表成功的1，则刷新整个页面，否则，提示错误。

然后我们看看Controller层的代码。

### 5.Controller层实现

首先我在application目录下的common.php文件定义了一个公共方法，用来返回json格式数据给View层。

```text
<?php
// 应用公共文件

function returnjson($arr_data){
    $arr = array(
        'code' => $arr_data['0'],
        'msg'  => $arr_data['1'],
        'data' => $arr_data['2']
    );
    if(!isset($arr_data['2'])){
        unset($arr['data']);
    }
    if(!isset($arr_data['1'])){
        unset($arr['msg']);
    }
    echo json_encode($arr);exit;
}
```

返回的数据有三个，code状态码,msg信息，data返回的数据。

然后看survey.php下的add\_qsn方法：

```text
 //新增
    public function add_qsn()
    {
        $data['create_name'] = input('post.create_name'); //thinkPHP中的助手函数,我们用ajax提交的数据
        $data['name'] = input('post.name');
        $data['time'] = input('post.time');
        $data['model_id'] = uniqid('model', true);//使用uniqid形成一个特定唯一的model_id
        if (empty($data['create_name'])) {//查询创建人是否为空
            returnjson([3, 'warning1', '']);
        }
        //查询model_id 是否重复 模板不存在的话 就能添加此条记录
        $chk_model_id = db('qsn_model')->where('create_name', $data['create_name'])->find($data);
        if (empty($chk_model_id)) {
            //插入数据库
            $insert = db('qsn_model')->insert($data);
            if ($insert) {
                returnjson([1, '添加成功', '']);
            } else {
                returnjson([2, '添加失败', '']);
            }
        } else {
            returnjson([3, '该数据已存在', '']);
        }
    }
```

我这里用了TP5提供的助手函数，熟悉PHP的话，应该会知道$\_GET和$\_POST,这里就是对应$\_POST,TP5对这两个函数形成了助手函数，防止一些SQL注入等安全因素带来的隐患。 此外，还有一个封装的returnjson方法，对我们返回的状态码，数据，msg进行规范操作。

```text
function returnjson($arr_data){
    $arr = array(
        'code' => $arr_data['0'],
        'msg'  => $arr_data['1'],
        'data' => $arr_data['2']
    );
    if(!isset($arr_data['2'])){
        unset($arr['data']);
    }
    if(!isset($arr_data['1'])){
        unset($arr['msg']);
    }
    echo json_encode($arr);exit;
}
```

### 6.功能一览

然后我们查看下我们的功能实现了没 ![1.gif](https://upload-images.jianshu.io/upload_images/9003228-b1ef6dff205a5c00.gif?imageMogr2/auto-orient/strip)

可以看见，列表的模板已经从2个增加到3个了。 大功告成啦，如果喜欢就给颗小💗💗吧~

此外还有对应的 [答题管理系统二:模板删除功能](https://www.jianshu.com/p/9f6f611b19fb)

[答题管理系统三:题目及选项增删功能](https://www.jianshu.com/p/599623baca26)

[答题管理系统四:答题功能](https://www.jianshu.com/p/f8d18ca91c6f)

[答题管理系统五:统计功能](https://www.jianshu.com/p/27bde9989f8a)

