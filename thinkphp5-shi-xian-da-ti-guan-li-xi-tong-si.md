# ThinkPHP5实现答题管理系统（四）

我们之前三篇文章实现了模板+题目+选项的增删，现在我们要完成答题管理系统的答题模块啦。

#### 一.UI及功能设计

既然是用户答题，就要有一张表专门用来存放用户答题的答案。我的设想是这样的，我们点击答题按钮的时候，就显示出第一题，判断是否是模板下最后一题，来显示名为next按钮的字体样式为下一题还是完成。并且，对后台传过来的qsn\_type进行区分，type=0的单选显示单选框，type=1的多选显示复选框，type=2的问答显示文本域，大概是如下效果。 ![&#x5355;&#x9009;.PNG](https://upload-images.jianshu.io/upload_images/9003228-0a923af45be94a4c.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![&#x591A;&#x9009;.PNG](https://upload-images.jianshu.io/upload_images/9003228-e623bb7d30cdc381.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![&#x95EE;&#x7B54;.PNG](https://upload-images.jianshu.io/upload_images/9003228-2d1435b884549d59.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

为此，next按钮要返回的json格式数据如下。 ![&#x6784;&#x5EFA;json.PNG](https://upload-images.jianshu.io/upload_images/9003228-a67437319591b2b7.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

用PHP构造一个二维关联数组，其中二维数组中，还有一个二维数组，第一索引表示题目的order\_num排序，第二索引表示option\_num选项内容以及order\_num选项排序号，最后将这个内层二维数组与别的参数使用PHP的array\_merge函数进行数组合并，返回给html页面。

#### 二.数据库表设计（psg\_qsn\_r表）

![psg\_qsn\_r&#x8868;.PNG](https://upload-images.jianshu.io/upload_images/9003228-e6148cd1e3a2f939.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

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
                var url = "{:url('survey/answer_list')}";
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
 public function answer_list()
    {//答题的首次加载
            $model_id = input('post.model_id/a'); //tp5提交数组用/a来实现
            if (empty($model_id)) {
                returnjson([3, '请选择模板答题', '']);
            }
        $str_device_id = implode(',', $model_id); //数组拆分
        $where['model_id'] = array('in', $str_device_id);
        $qsn_list = db('qsn')->where($where)->order('order_num')->field('content,qsn_id,order_num,model_id,qsn_type')->select(); //获取问题内容
        if (empty($qsn_list)) {
            returnjson([2, '暂无题目,无法答题', '']);
        }
        $bool_num = false;
        $max_order_num = db('qsn')->where($where)->max('order_num');
        if ($qsn_list[0]['order_num'] == $max_order_num) {
            $bool_num = true;
        }
        // $where1['qsn_id']=$qsn_list[0]['qsn_id'];
            // // $qsn_list[0]['qsn_id存在
            $option_list = db('qsn_detail')->where('qsn_id', $qsn_list[0]['qsn_id'])->field('option_num,order_num')->select(); //获取选项以及选项排序

        //array_merge()合成两个数组
        if ($option_list) {
            $return_list = array_merge($qsn_list[0], $option_list);
            $return_list['bool_num'] = $bool_num;
            returnjson([1, 'success', $return_list]);
        } else {//那就不是选项了 是问答题
            $return_list = $qsn_list[0];
            $return_list['bool_num'] = $bool_num;
            returnjson([1, '问答题', $return_list]);
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
                var url = "{:url('survey/next_qsn')}";
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
 //找出同一套模板下的order_num+1的题目
    public function next_qsn()
    {
        $model_id = input('post.model_id');
        $order_num = input('post.order_num');
        $choice = input('post.choice/a');
        $count_choice = count($choice);
        $str_choice1 = implode('', $choice);
        if (empty($str_choice1)) {
            returnjson([3, '请答题', $choice]);
        }
        $qsn_type = (int) input('post.qsn_type');
        $order_num = (int) $order_num;
        $qsn_list = db('qsn')->where('model_id', $model_id)->where('order_num', $order_num)->field('content,qsn_id,order_num,model_id,flight_id,qsn_type')->find(); //获取问题内容
        $psg_list['qsn_id'] = $qsn_list['qsn_id'];
        $psg_list['model_id'] = $qsn_list['model_id'];
        $psg_list['flight_id'] = $qsn_list['flight_id'];
        $psg_list['qsn_type'] = $qsn_list['qsn_type'];
        //这里要改，这里的detailID不是随机生成的 而是读取出对应题目的detail_id
        $option_list = db('qsn_detail')->where('qsn_id', $qsn_list['qsn_id'])->field('detail_id')->select();//这里是二维数组
        //这里的detail ID和date就随机生成了
        //这里 我们应该count出option_list数组的长度，然后循环赋给psg_list中的detail_id,保证psg_qsn表中的detailID唯一
        // returnjson([2, '恭喜您完成了答题', $option_list]);


        $str_choice = implode(',', $choice);
        for ($x = 0; $x < $count_choice; $x++) {
            $psg_list['detail_id'] = $option_list[$x]['detail_id'];
            $psg_list['psg_qsn_r_id'] = uniqid('psg_qsn_r_id', true);
            $psg_list['choose'] = $choice[$x];
            $psg_res = db('psg_qsn_r')->insert($psg_list);
        }
        //先要将当前问题插入psg_qsn表中 所以我们要先判断 然后看看能否插入psg_qsn表 否则对应的字段就是下一个的字段
        $order_num = $order_num + 1;
        //只查询一个数据 用find 否则返回的数据就是二维数组
        $qsn_list = db('qsn')->where('model_id', $model_id)->where('order_num', $order_num)->field('content,qsn_id,order_num,model_id,flight_id,qsn_type')->find(); //获取问题内容
         $bool_num = false;
        $max_order_num = db('qsn')->where('model_id', $model_id)->max('order_num');
        if ($qsn_list['order_num'] == $max_order_num) {
            $bool_num = true;
        }
        if (empty($qsn_list)) {
            returnjson([2, '恭喜您完成了答题', $choice]);
        }

        //qsn_list已经是一个数组了
        $option_list = db('qsn_detail')->where('qsn_id', $qsn_list['qsn_id'])->field('option_num,order_num')->select();

        //一个bool_num来判单是否是最后一题
        //
        if ($option_list) {
            $return_list = array_merge($qsn_list, $option_list);
            $return_list['bool_num'] = $bool_num;
            returnjson([1, 'success', $return_list]);
        } else {//是文本域
            $return_list = $qsn_list;
            $return_list['bool_num'] = $bool_num;
            returnjson([1, '问答', $return_list]);
        }
    }
```

然后我们看看最终的效果。

#### 四.效果一览

![1.gif](https://upload-images.jianshu.io/upload_images/9003228-4f0fe2f81bdd6cfa.gif?imageMogr2/auto-orient/strip)

OK，完成啦

