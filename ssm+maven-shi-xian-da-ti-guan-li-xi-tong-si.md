# SSM+maven实现答题管理系统（四）

我们之前三篇文章实现了模板+题目+选项的增删，现在我们要完成答题管理系统的答题模块啦。

#### 一.UI及功能设计

既然是用户答题，就要有一张表专门用来存放用户答题的答案。我的设想是这样的，我们点击答题按钮的时候，就显示出第一题，判断是否是模板下最后一题，来显示名为next按钮的字体样式为下一题还是完成。并且，对后台传过来的qsn\_type进行区分，type=0的单选显示单选框，type=1的多选显示复选框，type=2的问答显示文本域，大概是如下效果。 ![&#x5355;&#x9009;.PNG](https://upload-images.jianshu.io/upload_images/9003228-0a923af45be94a4c.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![&#x591A;&#x9009;.PNG](https://upload-images.jianshu.io/upload_images/9003228-e623bb7d30cdc381.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![&#x95EE;&#x7B54;.PNG](https://upload-images.jianshu.io/upload_images/9003228-2d1435b884549d59.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

为此，next按钮要返回的json格式数据如下。 ![&#x6784;&#x5EFA;json.PNG](https://upload-images.jianshu.io/upload_images/9003228-a67437319591b2b7.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

用PHP构造一个二维关联数组，其中二维数组中，还有一个二维数组，第一索引表示题目的order\_num排序，第二索引表示option\_num选项内容以及order\_num选项排序号，最后将这个内层二维数组与别的参数使用PHP的array\_merge函数进行数组合并，返回给html页面。

#### 二.数据库表设计（psg\_qsn\_r表）

![image.png](https://upload-images.jianshu.io/upload_images/9003228-6ffc63ec5ecc2bc3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

主要的就是choose对应用户选择的选项，1,2,3...对应A,B,C，如果是问答，存的就是一串字符串。

#### 三.代码实现

首先是answer答题按钮的点击监听:

```text
 $('#btn-answer').click(function() {
                // alert(id_array);
                layer.msg('加载中', {
                    icon: 16,
                    shade: 0.01,
                    time: '9999999'
                });
             var url = "survey/answer_list";
                var data = {
                    model_id: id_array //这里将当前的model_id数组传到后端
                }
                $.post(url, data, function(data) {
                    layer.close(layer.index); //关闭正在加载中弹出层
                    console.log(id_array);
                    if (data.code == 1) {
                        // alert(data.data[0].option_num);
                        if (data.data.bool_num == true) {
                            $('#next').text("完成");
                        }
                        $('#set-answer label[name="qsn_content1"]').css("width", "200px");
                        $('#set-answer label[name="qsn_content1"]').text(data.data.content);
                        if (data.data.qsn_type == 1) {
                            $.each(data.data, function(i, item) {
                                if (item.order_num == null) return false; //function中无法用break跳出 我们用return
                                //这边要将type也传过来 然后根据type来往anser_list中添加值

                                $('#answer_list').append(
                                    // ' <br/><label class="layui-form-label">' + item.order_num + ':</label>' +
                                    '<div class ="layui-input-block"><input type="checkbox" lay-skin="primary" name="checkbox[]" value="' + item.order_num + '" title="' + letter[item.order_num - 1] + ' : ' + item.option_num + '"></div>'

                                )
                            });
                            form.render('checkbox');
                        } else if (data.data.qsn_type == 0) {
                            $.each(data.data, function(i, item) {
                                if (item.order_num == null) return false; //function中无法用break跳出 我们用return
                                //这边要将type也传过来 然后根据type来往anser_list中添加值

                                $('#answer_list').append(
                                    // ' <br/><label class="layui-form-label">' + item.order_num + ':</label>' +
                                    '<div class ="layui-input-block"><input type="radio" lay-skin="primary" name="checkbox[]" value="' + item.order_num + '" title="' + letter[item.order_num - 1] + ' : ' + item.option_num + '"></div>'
                                )
                            });
                            form.render('radio');
                        } else if (data.data.qsn_type == 2) {

                            // if (item.order_num == null) return false; //function中无法用break跳出 我们用return
                            //这边要将type也传过来 然后根据type来往anser_list中添加值

                            $('#answer_list').append(
                                // ' <br/><label class="layui-form-label">' + item.order_num + ':</label>' +
                                ' <div class ="layui-input-block"><textarea name="text" id="text" required lay-verify="required" placeholder="请输入" class="layui-textarea"></textarea>'
                            )
                            form.render();

                        }



                        // alert(data.data.order_num);
                        getqsn_type(data.data.qsn_type);
                        getmodel_id(data.data.model_id);
                        getorder_num(data.data.order_num);
                        layer.open({
                            type: 1,
                            skin: 'layui-layer-rim', //加上边框
                            area: ['660px', '350px'], //宽高
                            content: $('#set-answer'),
                            cancel: function(index, layero) {
                                $('#answer_list').empty();
                                $('#next').text("下一题");
                                layer.close(index)
                                return false;
                            },
                            end: function() {
                                $('#answer_list').empty();
                                $('#next').text("下一题");
                                layer.closeAll();
                            }
                        });
                    } else {
                        layer.msg(data.msg, {
                            icon: 5
                        });
                    }
                }, "json");

            });
```

其中有对问题类型进行判别，并且对next样式作更改，然后是其对应的Controller层:

```text
 @RequestMapping(value="Index/survey/answer_list")
    @ResponseBody
    public Map<String,Object> firstAnswer(HttpServletRequest req) throws IOException{
        String[] arr_modelId = req.getParameterValues("modelId[]");
        Map map=new HashMap();
        if (arr_modelId.length==0)
            return api.returnJson(3,"请选择模板答题");
        String modelId=arr_modelId[0];
        List<Qsn> qsnList=qsnService.findQsnList(modelId);
        if (qsnList.isEmpty()){
            return api.returnJson(2,"暂无题目");
        }
        boolean bool_num=false;
        //判断这套模板的第一道题目是否是最后一题
        if(qsnList.get(0).getOrderNum()==qsnService.findMaxOrderNum(modelId))
        bool_num=true;

        //不管是不是最后一题，都查询选项
        List<Detail> optionList=detailService.findOptionList(qsnList.get(0).getQsnId());

        if (!qsnList.get(0).getQsnType().equals(2)){//单选/多选
            map.put("qsn_type",qsnList.get(0).getQsnType());
            map.put("content",qsnList.get(0).getContent());
            map.put("modelId",qsnList.get(0).getModelId());
            map.put("orderNum",qsnList.get(0).getOrderNum());
            map.put("qsnId",qsnList.get(0).getQsnId());
            map.put("bool_num",bool_num);
            for (int i = 0; i <optionList.size() ; i++) {
                map.put(i,optionList.get(i));
            }
            return  api.returnJson(1,"success",map);

        }else{//问答题
            map.put("qsn_type",qsnList.get(0).getQsnType());
            map.put("content",qsnList.get(0).getContent());
            map.put("modelId",qsnList.get(0).getModelId());
            map.put("orderNum",qsnList.get(0).getOrderNum());
            map.put("qsnId",qsnList.get(0).getQsnId());
            map.put("bool_num",bool_num);
            for (int i = 0; i <optionList.size() ; i++) {
                map.put(i,optionList.get(i));
            }
            return  api.returnJson(1,"success",map);
//            return  api.returnJson(2,"此路不通");
        }

    }
```

然后是View层next按钮的监听事件，跟这个差不多。

```text
 $('#next').click(function() {
                var qsn_type = t_qsn_type;

                var is_answered = false;
                if (qsn_type == 1) {
                    var count = $("#set-answer input[type='checkbox']:checked").length
                    if (count == 0) {
                        is_answered = true;
                    }
                    $("#set-answer input[type='checkbox']:checked").each(function() {
                        choice.push($(this).val());
                    });
                } else if (qsn_type == 0) {
                    var count = $("#set-answer input[type='radio']:checked").length
                    if (count == 0) {
                        is_answered = true;
                    }
                    $("#set-answer input[type='radio']:checked").each(function() {
                        choice.push($(this).val());
                    });
                } else if (qsn_type == 2) {
                    var count = $("#set-answer #text").length
                    if (count == 0) {
                        is_answered = true;
                    }
                    $("#set-answer #text").each(function() {
                        choice.push($(this).val());
                    });
                }
                if (is_answered == true) {
                    layer.msg('请选择答案');
                    return;
                }
                alert(choice);
                layer.msg('加载中', {
                    icon: 16,
                    shade: 0.01,
                    time: '9999999'
                });
                alert(choice);
              var url = "survey/next_qsn";
                var t_choice = choice;
                var model_id = t_model_id;
                var order_num = t_order_num;
                var data = {
                    model_id: model_id,
                    order_num: order_num,
                    choice: t_choice,
                    qsn_type: qsn_type
                }
                $.post(url, data, function(data) {
                    layer.close(layer.index); //关闭正在加载中弹出层
                    form.render(null, 'answer');
                    choice = [];
                    console.log(id_array);
                    if (data.code == 1) {
                        if (data.data.bool_num == true) {
                            $('#next').text("完成");
                        }
                        // alert(data.data[0].option_num);
                        $('#answer_list').empty();
                        $('#set-answer label[name="qsn_content1"]').text(data.data.content);
                        // alert(data.data.order_num);
                        if (data.data.qsn_type == 1) {
                            $.each(data.data, function(i, item) {
                                if (item.order_num == null) return false; //function中无法用break跳出 我们用return
                                $('#answer_list').append(
                                    // ' <br/><label class="layui-form-label">' + item.order_num + ':</label>' +
                                    '<div class ="layui-input-block"><input type="checkbox" lay-skin="primary" name="sex" value="' + item.order_num + '" title="' + letter[item.order_num - 1] + ' : ' + item.option_num + '"></div>'

                                )


                            });
                            form.render('checkbox');
                        } else if (data.data.qsn_type == 0) {
                            $.each(data.data, function(i, item) {
                                if (item.order_num == null) return false; //function中无法用break跳出 我们用return
                                //这边要将type也传过来 然后根据type来往anser_list中添加值

                                $('#answer_list').append(
                                    // ' <br/><label class="layui-form-label">' + item.order_num + ':</label>' +
                                    '<div class ="layui-input-block"><input type="radio" lay-skin="primary" name="checkbox[]" value="' + item.order_num + '" title="' + letter[item.order_num - 1] + ' : ' + item.option_num + '"></div>'
                                )
                            });
                            form.render('radio');
                        } else if (data.data.qsn_type == 2) {
                            $('#answer_list').append(
                                // ' <br/><label class="layui-form-label">' + item.order_num + ':</label>' +
                                ' <div class ="layui-input-block"><textarea name="text" id="text"  lay-verify="required" placeholder="请输入" class="layui-textarea"></textarea>'
                            )
                            form.render();

                        }
                        getqsn_type(data.data.qsn_type);
                        getmodel_id(data.data.model_id);
                        getorder_num(data.data.order_num);

                    } else if (data.code == 2) {
                        layer.msg(data.msg, {
                            icon: 6
                        });
                        setTimeout('layer.closeAll()', 2000);
                    } else if (data.code == 3) {
                        layer.msg(data.msg, {
                            icon: 5
                        });
                    }
                }, "json");
```

然后是Controller层代码。

```text
   //下一题 这题答题录入 并返回下一题数据
//    modelId
//    orderNum
//    choice//数组
//    qsn_type
    @RequestMapping(value="Index/survey/next_qsn")
    @ResponseBody
    public Map<String,Object> answerRecord(HttpServletRequest req) throws IOException{
        String modelId = req.getParameter("modelId");
        String str_orderNum = req.getParameter("orderNum");
        String qsnType = req.getParameter("qsn_type");
        String[] choose = req.getParameterValues("array[]");
        Map map=new HashMap();
        short orderNum=Short.parseShort(str_orderNum);
        if(choose.length==0)
            return api.returnJson(3,"请答题");
        Qsn qsn=qsnService.findByOrderNum(orderNum,modelId);
        Choose c=new Choose();
        c.setQsnId(qsn.getQsnId());
        c.setModelId(qsn.getModelId());
        c.setFlightId(qsn.getFlightId());
        c.setQsnType(qsn.getQsnType());

        if (!qsnType.equals(2)) {
            String str_choose = StringUtils.join(choose, ",");

            List<Detail> optionList = detailService.findOptionList(qsn.getQsnId());
            for (int i = 0; i < choose.length; i++) {
                c.setDetailId(optionList.get(i).getDetailId());
                c.setChoose(choose[i]);
                UUID uuid = UUID.randomUUID();
                String psgQsnId = uuid.toString();
                c.setPsgQsnRId(psgQsnId);
                int isInsert = chooseService.InsertChoose(c);
            }
        }else{//如果是2的话 那就是整个录入进去
            String str_choose=choose[0];
            List<Detail> optionList = detailService.findOptionList(qsn.getQsnId());
            c.setDetailId(optionList.get(0).getDetailId());
            c.setChoose(str_choose);
            UUID uuid = UUID.randomUUID();
            String psgQsnId = uuid.toString();
            c.setPsgQsnRId(psgQsnId);
            int isInsert = chooseService.InsertChoose(c);
        }

        //然后我们要返回下一题的数据 这里注意orderNum+=1和orderNum=orderNum+1
        short s=1;
        orderNum=(short) (orderNum+s);
        Qsn qsnNext=qsnService.findByOrderNum(orderNum,modelId);
        boolean bool_num=false;
        if (qsnNext==null){
            return  api.returnJson(2,"恭喜您完成了答题");
        }
        if (qsnNext.getOrderNum()==qsnService.findMaxOrderNum(modelId)){
            bool_num=true;
        }


        //然后我们查询选项
        List<Detail> optionList1=detailService.findOptionList(qsnNext.getQsnId());
        if (!qsnNext.getQsnType().equals(2)){//单选/多选
            map.put("qsn_type",qsnNext.getQsnType());
            map.put("content",qsnNext.getContent());
            map.put("modelId",qsnNext.getModelId());
            map.put("orderNum",qsnNext.getOrderNum());
            map.put("qsnId",qsnNext.getQsnId());
            map.put("bool_num",bool_num);
            for (int i = 0; i <optionList1.size() ; i++) {
                map.put(i,optionList1.get(i));
            }
            return  api.returnJson(1,"success",map);

        }else {//问答题
            map.put("qsn_type", qsnNext.getQsnType());
            map.put("content", qsnNext.getContent());
            map.put("modelId", qsnNext.getModelId());
            map.put("orderNum", qsnNext.getOrderNum());
            map.put("qsnId", qsnNext.getQsnId());
            map.put("bool_num", bool_num);
            for (int i = 0; i < optionList1.size(); i++) {
                map.put(i, optionList1.get(i));
            }
            return api.returnJson(1, "success", map);
        }
    }
```

除了上一节提供的qsn和detail的Service层代码 这里还需要chooseService的代码: chooseService：

```text
package com.sl.example.service;

import com.sl.example.pojo.Choose;

import java.util.List;

public interface ChooseService {
    public List<Choose> findChooseByModelId(String modelId);

    List<Choose> countDataBy2Id(String qsnId,String detailId);

    public int InsertChoose(Choose record);

    List<Choose> findByQsnId(String qsnId);
}
```

chooseServiceImpl:

```text
package com.sl.example.service;

import com.sl.example.dao.ChooseMapper;
import com.sl.example.dao.ModelMapper;
import com.sl.example.pojo.Choose;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;
import java.util.List;

@Service("chooseService")
@Transactional
public class ChooseServiceImpl implements ChooseService{

    @Resource
    private ChooseMapper chooseMapper;
    @Override
    public List<Choose> findChooseByModelId(String modelId) {
        return chooseMapper.selectByModelId(modelId);
    }

    @Override
    public List<Choose> countDataBy2Id(String qsnId, String detailId) {
        return chooseMapper.selectByQsnIdDetailId(qsnId,detailId);
    }

    @Override
    public int InsertChoose(Choose record) {
        return chooseMapper.insertSelective(record);
    }

    @Override
    public List<Choose> findByQsnId(String qsnId) {
        return chooseMapper.selectByQsnId(qsnId);
    }
}
```

然后我们看看最终的效果。

#### 四.效果一览

![1.gif](https://upload-images.jianshu.io/upload_images/9003228-4f0fe2f81bdd6cfa.gif?imageMogr2/auto-orient/strip)

OK，完成啦

