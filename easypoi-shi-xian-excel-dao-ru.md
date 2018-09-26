# easypoi实现Excel导入

最近做的一个项目用到了Excel导入，我选择了使用easypoi进行Excel解析。

#### 1.前期准备

如果使用maven等项目管理工具，在配置文件pom.xml中，添加以下三个依赖:

```text
  <dependency>
            <groupId>cn.afterturn</groupId>
            <artifactId>easypoi-base</artifactId>
            <version>3.0.3</version>
        </dependency>
        <dependency>
            <groupId>cn.afterturn</groupId>
            <artifactId>easypoi-web</artifactId>
            <version>3.0.3</version>
        </dependency>
        <dependency>
            <groupId>cn.afterturn</groupId>
            <artifactId>easypoi-annotation</artifactId>
            <version>3.0.3</version>
        </dependency>
```

如果没使用项目管理工具，在build path中导入三个jar包 ![image.png](https://upload-images.jianshu.io/upload_images/9003228-038f151efaf579f7.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

#### 2.业务流程

客户下载Excel模板-----&gt;客户填写Excel上传-----&gt;解析Excel-----&gt;将数据持久化存储（录入数据库）-----&gt;查询操作，将数据显示在数据表格

Excel模板: ![image.png](https://upload-images.jianshu.io/upload_images/9003228-1e93689d21a0a4f0.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

填写后的Excel: ![image.png](https://upload-images.jianshu.io/upload_images/9003228-267e7d0cd4492258.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

#### 3.效果预览

![layer.gif](https://upload-images.jianshu.io/upload_images/9003228-cc67aaee1fd93c51.gif?imageMogr2/auto-orient/strip)

可以看见，我们在Excel中录入的名称为hello和world的数据已经录入数据库了。

#### 4.代码实现

前端使用layui\(别的前端UI框架也可以 使用文件上传功能即可\):

```text
<div class="layui-btn-container">
    <a class="layui-btn btn-add btn-default" id="btn-excel">选择Excel文件</a>
    &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
    <a class="layui-btn btn-add btn-default" id="btn-excel-sure">上传导入</a>
</div>
  layui.use(['upload'], function() {

       var  $ = layui.jquery;

        var upload = layui.upload;
}
  layui.use(['element', 'form', 'table', 'layer', 'vip_table', 'laydate','upload'], function() {
        var form = layui.form,
            table = layui.table,
            layer = layui.layer,
            vipTable = layui.vip_table,
            element = layui.element,
            $ = layui.jquery;
        var laydate = layui.laydate;
        var upload = layui.upload;

   //Excel导入
        upload.render({
            elem: '#btn-excel'
            ,url: 'layer/excelparser?fileName=1'
            ,auto: false
            //,multiple: true
            ,bindAction: '#btn-excel-sure'
            ,size: 2048 //最大允许上传的文件大小 2M
            ,accept: 'file' //允许上传的文件类型
            ,exts:'xlsx'//只上传pdf文档
            ,done: function(res){
                console.log(res)
                if(res.code == 1){//成功的回调
                    //do something （比如将res返回的图片链接保存到表单的隐藏域）
                    // $('#set-add-put input[name="fileName"]').val(res.data.fileName);

                    layer.msg(res.msg, {
                        icon: 6
                    });
                    location.reload();
                }else if(res.code==2){
                    layer.msg(res.msg, {
                        icon: 5
                    });
                }
            }
        });
});
```

这里使用layui实现上传下载的组件，并对返回结果进行回调，状态码1成功，2失败

后台: Controller层:

```text
 /**
     * 处理Excel解析的方法
     * @param file 前台上传的文件对象
     * @return
     */
    @RequestMapping(value = "Index/layer/excelparser")
    @ResponseBody
    public   Map<String,Object> Excel(HttpServletRequest request,@RequestParam("file")MultipartFile file)throws Exception
    {
        Map<String, Object> dataMap = new HashMap<String, Object>();
        String fileName1 = request.getParameter("fileName");// 设置文件名，根据业务需要替换成要下载的文件名
        String fileName;
        try {
            //上传目录地址


            String uploadDir = request.getSession().getServletContext().getRealPath("/") +"upload/";

            uploadDir=uploadDir.substring(0,uploadDir.length()-1);
            uploadDir=uploadDir+"\\";//下载目录
            String realPath=uploadDir+fileName1;//
            File dir = new File(realPath);
            if(!dir.exists())
            {
                dir.mkdir();
            }
            //调用上传方法
            fileName=upload.executeUpload1(uploadDir, file,fileName1);
            uploadDir=uploadDir.substring(0,uploadDir.length()-1);
            dataMap.put("fileName",fileName);
            dataMap.put("dir",uploadDir);
        }catch (Exception e)
        {
            //打印错误堆栈信息
            e.printStackTrace();
            return api.returnJson(2,"解析失败",dataMap);
        }
        ExcelParser(fileName);
        return api.returnJson(1,"解析成功",dataMap);
    }
 public void ExcelParser(String fileName)throws Exception{
        ImportParams params = new ImportParams();
        params.setTitleRows(1);
        params.setHeadRows(1);
        long start = new Date().getTime();
        List<Layer> list=new ArrayList<>();
        list = upload.importExcel("C:/Users/sl/Desktop/layer/layer/src/main/webapp/upload/"+fileName, 1, 1, Layer.class);
        System.out.println(new Date().getTime() - start);
        System.out.println(list.size());
        System.out.println(list);
        int testId=1;
        int isInsert=0;
        for (int i = 0; i <list.size() ; i++) {
            Layer layer=new Layer();
            UUID uuid=UUID.randomUUID();
            String layerId=uuid.toString();
            layer.setLayerId(layerId);
            layer.setLayerName(list.get(i).getLayerName());
            layer.setDescription(list.get(i).getDescription());
            layer.setRecordTime(list.get(i).getRecordTime());
            layer.setReleaseTime(list.get(i).getReleaseTime());
            int is_add=layerService.InsertLayer(layer);
            System.out.println(is_add);
        }
}
```

用到的工具类:Upload.java

```text
package com.example.sl.layer.util;

import cn.afterturn.easypoi.excel.ExcelImportUtil;
import cn.afterturn.easypoi.excel.entity.ImportParams;
import org.apache.commons.lang3.StringUtils;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.util.List;
import java.util.UUID;

//上传
public class Upload {

    public  String executeUpload1(String uploadDir,MultipartFile file,String fileName) throws Exception
    {
        //文件后缀名
        String suffix = file.getOriginalFilename().substring(file.getOriginalFilename().lastIndexOf("."));
        //上传文件名
        String filename = fileName + suffix;
        //服务器端保存的文件对象
        File serverFile = new File(uploadDir + filename);
        //将上传的文件写入到服务器端文件内
        file.transferTo(serverFile);

        return filename;
    }

    public  <T> List<T> importExcel(String filePath, Integer titleRows, Integer headerRows, Class<T> pojoClass){
        if (StringUtils.isBlank(filePath)){
            return null;
        }
        ImportParams params = new ImportParams();
        params.setTitleRows(titleRows);
        params.setHeadRows(headerRows);
        List<T> list = null;
        try {
            list = ExcelImportUtil.importExcel(new File(filePath), pojoClass, params);
        }catch (Exception e) {
            e.printStackTrace();

        }
        return list;
    }
    public  <T> List<T> importExcel(MultipartFile file, Integer titleRows, Integer headerRows, Class<T> pojoClass){
        if (file == null){
            return null;
        }
        ImportParams params = new ImportParams();
        params.setTitleRows(titleRows);
        params.setHeadRows(headerRows);
        List<T> list = null;
        try {
            list = ExcelImportUtil.importExcel(file.getInputStream(), pojoClass, params);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return list;
    }

}
```

model层实体类:\(这里使用easypoi的注解解析，对时间等特殊格式也加上解析即可\)

```text
package com.example.sl.layer.model;

import cn.afterturn.easypoi.excel.annotation.Excel;
import cn.afterturn.easypoi.excel.annotation.ExcelTarget;
import com.fasterxml.jackson.annotation.JsonFormat;

import java.util.Date;

@ExcelTarget("Layer")
public class Layer {
    private String layerId;

    @Excel(name = "法规名称", isImportField = "true_st")
    private String layerName;

    @Excel(name = "法规描述", isImportField = "true_st")
    private String description;



    @Excel(name = "法规发布日期",importFormat = "yyyy-MM-dd")
    private Date releaseTime;



    @Excel(name = "法规上传日期",importFormat = "yyyy-MM-dd")
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


    public void setReleaseTime(Date releaseTime) {
        this.releaseTime = releaseTime;
    }

    public Date getRecordTime() {
        return recordTime;
    }


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

dao层:

```text
package com.example.sl.layer.dao;

import com.example.sl.layer.model.Layer;
import org.apache.ibatis.annotations.Param;

import java.util.Date;
import java.util.List;

public interface LayerMapper {
    int insert(Layer record);
}
```

Service层: LayerService:

```text
package com.example.sl.layer.service;

import com.example.sl.layer.model.Layer;

import java.util.Date;
import java.util.List;

public interface LayerService {


    public int InsertLayer(Layer layer);


}
```

LayerServiceImpl:

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
    public int InsertLayer(Layer layer) {
        return layerMapper.insert(layer);
    }
}
```

#### 5.最终实现

![layer.gif](https://upload-images.jianshu.io/upload_images/9003228-cc67aaee1fd93c51.gif?imageMogr2/auto-orient/strip)

![image.png](https://upload-images.jianshu.io/upload_images/9003228-4ddc17032d82c12a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/9003228-af167faeee89db4b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

解析成功，并且入库。

喜欢就给颗小💗💗吧。

