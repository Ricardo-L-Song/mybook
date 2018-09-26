# SpringBoot实现上传下载\(一\)

最近在学Springboot相关知识，这次用Springboot做了一个上传下载的功能，[项目demo](https://github.com/Ricardo-L-Song/upAndDown) 上传一个法律名及其发布的年份等信息，然后还要能上传一个pdf文件\(这里限制下上传的后缀名就可以\)，上传之后，点击操作中的下载，下载对应的pdf文件。

预览: ![image.png](https://upload-images.jianshu.io/upload_images/9003228-23d52079d48a1b18.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9003228-8a78b0848a72f1d0.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

### 1.前期准备

编辑器:IDEA/eclipse/myeclipse layui文档的bang助:[layui开发使用文档](http://www.layui.com/doc/)

[如何使用IDEA创建Springboot项目](https://blog.csdn.net/u013777094/article/details/78580710/)

[使用使用IDEA进行资源标记](https://blog.csdn.net/xiaohei_neko/article/details/79353605)

根据项目结构图: ![image.png](https://upload-images.jianshu.io/upload_images/9003228-5325100cfc74e7dc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

标记java文件夹为Source root resources为resources root 为了迎合SSM框架的习惯 创建web文件夹 ![image.png](https://upload-images.jianshu.io/upload_images/9003228-47fc6f577ea7f52b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240) 配置web ![image.png](https://upload-images.jianshu.io/upload_images/9003228-ac3e18d72087ab9a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

详细的pom.xml等配置可以参见我的项目目录 这里不一一展开了 主要说一下各个目录文件夹的作用。

resources目录下的application.properties是spring相关配置文件夹 generatorConfig是逆向工程的配置文件 webapp目录下 public是JS/json/css等静态资源 upload是上传文件存放的文件夹 WEB-INF是存放jsp等渲染的网页模板文件的文件夹

### 2.架构设计\(MVC\)

![image.png](https://upload-images.jianshu.io/upload_images/9003228-5325100cfc74e7dc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

Model:dao+service+model三个包实现数据库连接以及使用数据库实现持久化存储

View:webapp包下的jsp等网页渲染文件

C:controller包下的多个控制器

### 3.数据库设计

![image.png](https://upload-images.jianshu.io/upload_images/9003228-586766f672625b15.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

没什么好说的 想说的都在图里

### 4.Code

首先实现上传功能。实现思路:使用layui提供给我们的upload Element。我们上传这个文件，限制后缀名为pdf，限制上传文件最大2048KB等。后台则将上传的File文件使用java的file.transferTo录入upload包下。

View层实现: layer.jsp：

```text
   <div class="layui-form-item">
            <div class="layui-input-block">
                <div class="layui-upload">
                    <button type="button" class="layui-btn layui-btn-normal" id="test8">选择文件</button>
                    <button type="button" class="layui-btn" id="upload">开始上传</button>
                    <input type="hidden" name="fileName" id="fileName">
                </div>
            </div>
        </div>
```

以上控件组件

```text
   layui.use(['element', 'form', 'table', 'layer', 'vip_table', 'laydate','upload'], function() {
        var form = layui.form,
            table = layui.table,
            layer = layui.layer,
            vipTable = layui.vip_table,
            element = layui.element,
            $ = layui.jquery;
        var laydate = layui.laydate;
        var upload = layui.upload;

 //上传
        upload.render({
            elem: '#test8'
            ,url: 'layer/upload'
            ,auto: false
            //,multiple: true
            ,bindAction: '#upload'
            ,size: 2048 //最大允许上传的文件大小 2M
            ,accept: 'file' //允许上传的文件类型
            ,exts:'pdf'//只上传pdf文档
            ,done: function(res){
                console.log(res)
                if(res.code == 1){//成功的回调
                    //do something （比如将res返回的图片链接保存到表单的隐藏域）
                    $('#set-add-put input[name="fileName"]').val(res.data.fileName);
                    $('#upload').hide();
                    layer.msg(res.msg, {
                        icon: 6
                    });
                }else if(res.code==2){
                    layer.msg(res.msg, {
                        icon: 5
                    });
                }
            }
        });
```

以上对upload的监听,并且对返回的状态码1（成功）,2（失败）做相应处理。

Controller层: LayerController：

```text
 /**
     * 处理上传文件方法请求
     * @param file 前台上传的文件对象
     * @return
     */
    @RequestMapping(value = "Index/layer/upload",method = RequestMethod.POST)
    @ResponseBody
    public  Map<String,Object> uploadOne(HttpServletRequest request,@RequestParam("file")MultipartFile file)
    {
       Map map=new HashMap();
        try {
            //上传目录地址
//            String uploadDir = request.getSession().getServletContext().getRealPath("upload");
            String uploadDir = request.getSession().getServletContext().getRealPath("/") +"upload/";
            //如果目录不存在，自动创建文件夹
            File dir = new File(uploadDir);
            if(!dir.exists())
            {
                dir.mkdir();
            }
            //调用上传方法
           String fileName=upload.executeUpload(uploadDir,file);
            uploadDir=uploadDir.substring(0,uploadDir.length()-1);
            map.put("fileName",fileName);
            map.put("dir",uploadDir);
        }catch (Exception e)
        {
            //打印错误堆栈信息
            e.printStackTrace();
            return api.returnJson(2,"上传失败",map);
        }

        return api.returnJson(1,"上传成功",map);
    }
```

相关的工具类: Api.java：

```text
package com.example.sl.layer.util;

import com.example.sl.layer.model.Layer;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

//接口
//code=1 success 2 fail 3 warning
public class Api {
    public Map<String,Object> returnJson(int code, String msg){
        Map map=new HashMap();
        map.put("code",code);
        map.put("msg",msg);
        return map;
    }

    public Map<String,Object> returnJson(int code, String msg, List<Map> data){
        Map map=new HashMap();
        map.put("code",code);
        map.put("msg",msg);
        map.put("data",data);
        return map;
    }

    public Map<String,Object> returnJson(int code, String msg, Map data){
        Map map=new HashMap();
        map.put("code",code);
        map.put("msg",msg);
        map.put("data",data);
        return map;
    }

    public Map<String,Object> returnJson(int code, String msg, int count,List<Layer> data){
        Map map=new HashMap();
        map.put("code",code);
        map.put("msg",msg);
        map.put("count",count);
        map.put("data",data);
        return map;
    }
}
```

Upload.java：

```text
package com.example.sl.layer.util;

import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.util.UUID;

//上传
public class Upload {
    /**
     * 提取上传方法为公共方法
     * @param uploadDir 上传文件目录
     * @param file 上传对象
     * @throws Exception
     */
    public String executeUpload(String uploadDir,MultipartFile file) throws Exception
    {
        //文件后缀名
        String suffix = file.getOriginalFilename().substring(file.getOriginalFilename().lastIndexOf("."));
        //上传文件名
        String filename = UUID.randomUUID() + suffix;
        //服务器端保存的文件对象
        File serverFile = new File(uploadDir + filename);
        //将上传的文件写入到服务器端文件内
        file.transferTo(serverFile);

        return filename;
    }
}
```

顺带一提，这里返回给前端的数据有:fileName文件名还有dir文件路径，之后做下载的时候会用到。

Model层: model包下的Layer.java实体类:

```text
package com.example.sl.layer.model;

import com.fasterxml.jackson.annotation.JsonFormat;

import java.util.Date;

public class Layer {
    private String layerId;

    private String layerName;

    private String description;

    @JsonFormat(pattern="yyyy",timezone="GMT+8")
    private Date releaseTime;

    @JsonFormat(pattern="yyyy",timezone="GMT+8")
    private Date recordTime;

    private String fileName;

    public String getLayerId() {
        return layerId;
    }

    public void setLayerId(String layerId) {
        this.layerId = layerId == null ? null : layerId.trim();
    }

    public String getLayerName() {
        return layerName;
    }


    public void setLayerName(String layerName) {
        this.layerName = layerName == null ? null : layerName.trim();
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description == null ? null : description.trim();
    }

    public Date getReleaseTime() {
        return releaseTime;
    }

    @JsonFormat(pattern="yyyy",timezone="GMT+8")
    public void setReleaseTime(Date releaseTime) {
        this.releaseTime = releaseTime;
    }

    public Date getRecordTime() {
        return recordTime;
    }

    @JsonFormat(pattern="yyyy",timezone="GMT+8")
    public void setRecordTime(Date recordTime) {
        this.recordTime = recordTime;
    }

    public String getFileName() {
        return fileName;
    }

    public void setFileName(String fileName) {
        this.fileName = fileName == null ? null : fileName.trim();
    }
}
```

dao包:LayerMapper：

```text
package com.example.sl.layer.dao;

import com.example.sl.layer.model.Layer;
import org.apache.ibatis.annotations.Param;

import java.util.Date;
import java.util.List;

public interface LayerMapper {
    int deleteByPrimaryKey(String layerId);

    int insert(Layer record);

    Layer selectByPrimaryKey(String layerId);

    List<Layer> selectAll();

    Layer selectByLayerName(String layerName);

    List<Layer> selectByDescription(String description);

    int updateByPrimaryKey(Layer record);

    List<Layer> selectByTime(@Param("time1")Date releaseTime1,@Param("time2") Date releaseTime2);
}
```

service包: LayerService：

```text
package com.example.sl.layer.service;

import com.example.sl.layer.model.Layer;

import java.util.Date;
import java.util.List;

public interface LayerService {
    public List<Layer> findAllLayers();

    public int InsertLayer(Layer layer);

    public int deleteLayer(String layerId);

    public Layer findByLayerName(String layerName);

    public List<Layer> findByDescription(String description);

    public List<Layer> findByTime(Date time1,Date time2);
}
```

LayerServiceImpl：

```text
package com.example.sl.layer.service;

import com.example.sl.layer.dao.LayerMapper;
import com.example.sl.layer.model.Layer;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;
import java.util.Date;
import java.util.List;

@Service("layerService")
@Transactional
public class LayerServiceImpl implements LayerService{

    @Resource
    private LayerMapper layerMapper;
    @Override
    public List<Layer> findAllLayers() {
        return layerMapper.selectAll();
    }

    @Override
    public int InsertLayer(Layer layer) {
        return layerMapper.insert(layer);
    }

    @Override
    public int deleteLayer(String layerId) {
        return layerMapper.deleteByPrimaryKey(layerId);
    }

    @Override
    public Layer findByLayerName(String layerName) {
        return layerMapper.selectByLayerName(layerName);
    }

    @Override
    public List<Layer> findByDescription(String description) {
        return layerMapper.selectByDescription(description);
    }

    @Override
    public List<Layer> findByTime(Date time1, Date time2) {
        return layerMapper.selectByTime(time1,time2);
    }
}
```

这里使用了jackson作date解析 所以实体类上有注解

### 5.效果一览

以上就完成了上传功能，让我们来看看效果

![&#x4E0A;&#x4F20;.gif](https://upload-images.jianshu.io/upload_images/9003228-cb79ca6111171601.gif?imageMogr2/auto-orient/strip)

帮助到你了就给颗小💗💗吧~

另附: [SpringBoot实现上传下载\(二\)](https://www.jianshu.com/p/2f1f8a7a46de)

