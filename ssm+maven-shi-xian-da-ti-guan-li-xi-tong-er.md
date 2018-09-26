# SSM+maven实现答题管理系统（二）

好的，现在我们来实现删除模板\(model表删除记录\)的功能，删除功能不难做，主要我们这次实现的是批量删除功能。

#### 一.思路整理

**首先我们删除模板是根据模板的id来删除的，不管是单项删除还是批量删除，我们这里使用layui获取当前行的功能就能获取模板的id**

**其次不管单项删除还是批量删除，只要将id放入数组中，后端对数组进行遍历，就能达到单项删除/批量删除**

#### 二.代码实现

思路实现以后，我们上代码。

**View层:**

```text
 var id_array = []; //获取选中行
            // 获取选中行 这里我们可以选择删除 这里的id_array是一个隐藏的值 device对应device_id user对应code survry对应model_id
            table.on('checkbox(test)', function(obj) { //监听复选框
                if (obj.type == 'all') {
                    if (obj.checked == true) {
                        var data = table.cache.dataCheck; //批量操作的表格复选框
                        id_array = [];
                        for (var l = 0; l < data.length; l++) {
                            id_array.push(data[l].model_id);
                        }
                        console.log(id_array);
                    } else {
                        id_array = [];
                    }
                } else {
                    if (obj.checked == true) {
                        id_array.push(obj.data.model_id);
                        console.log(id_array);
                    } else {
                        var index = id_array.indexOf(obj.data.model_id);
                        id_array.splice(index, 1);
                        console.log(id_array);
                    }
                }
            });

            $('#btn-delete-all').click(function() { //删除全部通过一个获取选中行的值，来删除
                layer.confirm('您确定要删除这些数据吗？', function(index) {
                    //打开正在加载中弹出层
                    layer.msg('加载中', {
                        icon: 16,
                        shade: 0.01,
                        time: '9999999'
                    });
                   var url = "survey/del_model";
                    var data = {
                        model_id: id_array //这里将当前的model_id传到后端
                    }
                    $.post(url, data, function(data) {
                        layer.close(layer.index); //关闭正在加载中弹出层
                        console.log(id_array);
                        if (data.code == 1) {
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
                });

            });
```

首先对数据表格进行监听，如果数据表格的当前行被选中，则往id\_array数组中push一个id，有多少个，push多少个，而后我们将数组用ajax上传，键名为model\_id.

**Controller层:**

```text
    //返回数据用responseBody
    //删除model
    @RequestMapping(value="Index/survey/del_model")
    @ResponseBody
    public Map<String,Object> delModel(HttpServletRequest req) throws IOException {
        String[] arr = req.getParameterValues("model_id[]");//前端传来的modelId
        int code;
        String msg;

//        if(arr!=null)
//        return api.returnJson(3,arr[0]);
        for (int i = 0; i < arr.length; i++) {
            List<Choose> is_answer=chooseService.findChooseByModelId(arr[i]);
            if (!is_answer.isEmpty()) {
                return api.returnJson(3,"抱歉，题目已经被作答，无法删除");
            }
            continue;
        }
        //如果没有 那么当前选中模板的题目都没有被答过 作级联删除 删除模板表 删除题目表 删除选项表(根据model查出Qsn 再根据qsnId删除Option)
        int is_del=modelService.deleteModelByIds(arr);//删除模板表
        for (int i = 0; i <arr.length ; i++) {
            int is_del_qsn=qsnService.deleteModel2Qsn(arr[i]);//删除题目表
            List<Qsn> qsnList=qsnService.findQsnList(arr[i]);//得到题目
            System.out.println(qsnList);
            for (int j = 0; j <qsnList.size() ; j++) {//删除选项表
                int isDelOptions=detailService.deleteOptionsByQsnId(qsnList.get(j).getQsnId());
            }
        }

        //先查询模板下的题目list 然后遍历list，根据item.id 即qsnId来删除
//        String[] is_del_option 还缺删除选项表的逻辑
        if (is_del!=0){
            code=1;
            msg="success";
        }else {
            code=2;
            msg="fail";
        }

        return api.returnJson(code,msg);
    }
```

这里进行说明一下，区别于tp5，我们使用httpservlet.request的getParameterValues\(\)方法来获取前端传入的array数组

Service层: modelService：

```text
package com.sl.example.service;


import com.sl.example.pojo.Model;

import java.util.List;

public interface ModelService {
    public List<Model> findAllModel();

    public int deleteModelById(String modelId);

    public int deleteModelByIds(String[] arr);

    public int InsertModel(Model model);

    public Model selectModelById(String modelId);
}
```

modelServiceImpl：

```text
package com.sl.example.service;

import com.sl.example.dao.ModelMapper;
import com.sl.example.pojo.Model;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;
import java.util.List;

@Service("modelService")
@Transactional
public class ModelServiceImpl implements ModelService{

    @Resource
    private ModelMapper modelMapper;

    @Override
    public List<Model> findAllModel() {
        return modelMapper.selectAllModel();
    }

    @Override
    public int deleteModelById(String modelId) {
        return modelMapper.deleteByPrimaryKey(modelId);
    }

    @Override
    public int deleteModelByIds(String[] arr) {
        return modelMapper.deleteByIds(arr);
    }

    @Override
    public int InsertModel(Model model) {
        return modelMapper.insertSelective(model);
    }

    @Override
    public Model selectModelById(String modelId) {
        return modelMapper.selectByPrimaryKey(modelId);
    }
}
```

将DAO层也贴出来:

```text
package com.sl.example.dao;

import com.sl.example.pojo.Model;

import java.util.List;

public interface ModelMapper {
    int deleteByPrimaryKey(String modelId);

    int deleteByIds(String[] list);

    int insert(Model record);

    int insertSelective(Model record);

    Model selectByPrimaryKey(String modelId);

    List<Model> selectAllModel();

    int updateByPrimaryKeySelective(Model record);

    int updateByPrimaryKey(Model record);
}
```

对应的ModelMapper.xml:

```text
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.sl.example.dao.ModelMapper" >
  <resultMap id="BaseResultMap" type="com.sl.example.pojo.Model" >
    <constructor >
      <idArg column="model_id" jdbcType="VARCHAR" javaType="java.lang.String" />
      <arg column="name" jdbcType="VARCHAR" javaType="java.lang.String" />
      <arg column="time" jdbcType="DATE" javaType="java.util.Date" />
      <arg column="create_name" jdbcType="VARCHAR" javaType="java.lang.String" />
      <arg column="rmk2" jdbcType="VARCHAR" javaType="java.lang.String" />
      <arg column="rmk3" jdbcType="VARCHAR" javaType="java.lang.String" />
      <arg column="rmk4" jdbcType="VARCHAR" javaType="java.lang.String" />
      <arg column="rmk5" jdbcType="VARCHAR" javaType="java.lang.String" />
    </constructor>
  </resultMap>
  <sql id="Base_Column_List" >
    model_id, name, time, create_name, rmk2, rmk3, rmk4, rmk5
  </sql>
  <select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="java.lang.String" >
    select 
    <include refid="Base_Column_List" />
    from t_gr_qsn_model
    where model_id = #{modelId,jdbcType=VARCHAR}
  </select>
  <select id="selectAllModel" resultMap="BaseResultMap" resultType="java.util.List">
    select
    <include refid="Base_Column_List" />
    from t_gr_qsn_model
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.String" >
    delete from t_gr_qsn_model
    where model_id = #{modelId,jdbcType=VARCHAR}
  </delete>
  <delete id="deleteByIds" parameterType="java.util.Arrays">
    delete from t_gr_qsn_model
    where model_id in
    <foreach collection="array" index="index" item="item" open="(" separator="," close=")">
      #{item}
    </foreach>
  </delete>
  <insert id="insert" parameterType="com.sl.example.pojo.Model" >
    insert into t_gr_qsn_model (model_id, name, time, 
      create_name, rmk2, rmk3, 
      rmk4, rmk5)
    values (#{modelId,jdbcType=VARCHAR}, #{name,jdbcType=VARCHAR}, #{time,jdbcType=DATE}, 
      #{createName,jdbcType=VARCHAR}, #{rmk2,jdbcType=VARCHAR}, #{rmk3,jdbcType=VARCHAR}, 
      #{rmk4,jdbcType=VARCHAR}, #{rmk5,jdbcType=VARCHAR})
  </insert>
  <insert id="insertSelective" parameterType="com.sl.example.pojo.Model" >
    insert into t_gr_qsn_model
    <trim prefix="(" suffix=")" suffixOverrides="," >
      <if test="modelId != null" >
        model_id,
      </if>
      <if test="name != null" >
        name,
      </if>
      <if test="time != null" >
        time,
      </if>
      <if test="createName != null" >
        create_name,
      </if>
      <if test="rmk2 != null" >
        rmk2,
      </if>
      <if test="rmk3 != null" >
        rmk3,
      </if>
      <if test="rmk4 != null" >
        rmk4,
      </if>
      <if test="rmk5 != null" >
        rmk5,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides="," >
      <if test="modelId != null" >
        #{modelId,jdbcType=VARCHAR},
      </if>
      <if test="name != null" >
        #{name,jdbcType=VARCHAR},
      </if>
      <if test="time != null" >
        #{time,jdbcType=DATE},
      </if>
      <if test="createName != null" >
        #{createName,jdbcType=VARCHAR},
      </if>
      <if test="rmk2 != null" >
        #{rmk2,jdbcType=VARCHAR},
      </if>
      <if test="rmk3 != null" >
        #{rmk3,jdbcType=VARCHAR},
      </if>
      <if test="rmk4 != null" >
        #{rmk4,jdbcType=VARCHAR},
      </if>
      <if test="rmk5 != null" >
        #{rmk5,jdbcType=VARCHAR},
      </if>
    </trim>
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="com.sl.example.pojo.Model" >
    update t_gr_qsn_model
    <set >
      <if test="name != null" >
        name = #{name,jdbcType=VARCHAR},
      </if>
      <if test="time != null" >
        time = #{time,jdbcType=DATE},
      </if>
      <if test="createName != null" >
        create_name = #{createName,jdbcType=VARCHAR},
      </if>
      <if test="rmk2 != null" >
        rmk2 = #{rmk2,jdbcType=VARCHAR},
      </if>
      <if test="rmk3 != null" >
        rmk3 = #{rmk3,jdbcType=VARCHAR},
      </if>
      <if test="rmk4 != null" >
        rmk4 = #{rmk4,jdbcType=VARCHAR},
      </if>
      <if test="rmk5 != null" >
        rmk5 = #{rmk5,jdbcType=VARCHAR},
      </if>
    </set>
    where model_id = #{modelId,jdbcType=VARCHAR}
  </update>
  <update id="updateByPrimaryKey" parameterType="com.sl.example.pojo.Model" >
    update t_gr_qsn_model
    set name = #{name,jdbcType=VARCHAR},
      time = #{time,jdbcType=DATE},
      create_name = #{createName,jdbcType=VARCHAR},
      rmk2 = #{rmk2,jdbcType=VARCHAR},
      rmk3 = #{rmk3,jdbcType=VARCHAR},
      rmk4 = #{rmk4,jdbcType=VARCHAR},
      rmk5 = #{rmk5,jdbcType=VARCHAR}
    where model_id = #{modelId,jdbcType=VARCHAR}
  </update>
</mapper>
```

我们来试验一下。 ![1.gif](https://upload-images.jianshu.io/upload_images/9003228-1628fad03209d674.gif?imageMogr2/auto-orient/strip)

成功啦，下一节我们讲对应某一套题目模板下的题目的增删。

