# Jquery解析Json对象/数组（含demo）

上班过程中，一直使用封装好的前端框架解析Json对象，这次我们用Jquery原生的解析来解析Json对象和Json数组。

#### 一.前期准备:

两个Json文件: json1:用来解析简单的Json对象

```text
{
    "comments": [{
        "content": "Hello",
        "id": 1,
        "nickname": "小三"
    }, {
        "content": "World",
        "id": 2,
        "nickname": "小四"
    }]
}
```

json2：用来解析Json数组

```text
{
    "comments": [{
        "content": "Hello",
        "id": 1,
        "nickname": "小三"
    }, {
        "content": "World",
        "id": 2,
        "nickname": "小四"
    }],

    "content": "我是Json对象content， 哈哈。",
    "infomap": {
        "性别": "男",
        "职业": "程序员",
        "博客": "https://www.jianshu.com/u/86890c00216d"
    },
    "title": "你好，世界"
}
```

然后一个编辑器（我用的是Hbuilder）,将解析的两个页面与两个json文件放入同一目录，方便解析，如下: ![Jq&#x89E3;&#x6790;.JPG](https://upload-images.jianshu.io/upload_images/9003228-a3222af23a15e73e.JPG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

#### 二.代码实现

**json1用来解析简单的json对象:**

```text
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title></title>
        <script src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js"></script>
        <script>
            function loadInfo(){
                $.getJSON("json1.json",function(data){
                    $("#info").html("")//清空info内容
                    $.each(data.comments,function(i,item){
                        $("#info").append(
                            "<div>"+i+item.id+"</div>"+
                            "<div>"+i+item.nickname+"</div>"+
                            "<div>"+i+item.content+"</div>"
                        );
                    });
                });
            }
        </script>
    </head>
    <body>
        <p id="info"></p>
        <button onclick="loadInfo()">点我加载数据</button>
        <!--{
    "comments": [{
        "content": "很不错嘛",
        "id": 1,
        "nickname": "纳尼"
    }, {
        "content": "哟西哟西",
        "id": 2,
        "nickname": "小强"
    }]
}-->
    </body>
</html>
```

然后在Hbuilder上运行这个网页看看: ![json1](https://upload-images.jianshu.io/upload_images/9003228-83d4c44c484e408f.gif?imageMogr2/auto-orient/strip)

**json2用来解析json数组:**

```text
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title></title>
        <script src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js"></script>
        <script>
            function loadInfo(){
                $.getJSON("json2.json",function(data){
                    $("#info").html("")//清空info内容

                    //JQ解析数组
                    $.each(data.comments,function(i,item){
                        $("#info").append(
                            "<div>"+item.id+"</div>"+
                            "<div>"+item.nickname+"</div>"+
                            "<div>"+item.content+"</div>"
                        );
                    });

                    //JQ解析单节点（多元素）（非数组）
                    $.each(data.infomap,function(i,item){//键值都解析出来
                        $("#mapinfo").append(i+"-------"+item+"<br />");
                    });

                    //JQ解析单节点（单元素） 直接使用data.解析 data作为所有的节点的根节点
                    $("#title").append(data.title+"br /");
                    $("#content").append(data.content);
                });
            }
        </script>
    </head>
    <body>
        <p id="info"></p>
        <p id="mapinfo"></p>
        <p id="title"></p>
        <p id="content"></p>
        <button onclick="loadInfo()">点我加载数据</button>
    </body>
</html>
```

同样运行一下这个html文件: ![json2.gif](https://upload-images.jianshu.io/upload_images/9003228-31daf09e3ab84cf0.gif?imageMogr2/auto-orient/strip)

以上。喜欢就给颗小💗💗吧~

