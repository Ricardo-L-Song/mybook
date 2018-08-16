# ThinkPHP5实现答题管理系统（五）

今天这篇文章是我们这个TP系统系列的最后一篇，主要是用来实现统计题目答案功能的实现。

#### 一.UI设计及功能实现

![&#x7EDF;&#x8BA1;1.PNG](https://upload-images.jianshu.io/upload_images/9003228-0bf0086e3696f062.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![&#x7EDF;&#x8BA1;.PNG](https://upload-images.jianshu.io/upload_images/9003228-57aad6653b6176e3.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

功能实现:乍一看，我们就知道这是实现了连表查询，点击统计按钮，我们要查询的是qsn表，qsn_detail表以及psg\_qsn_表，分别对应题目表，选项表以及答案表。而我们要提交到后端的，就是查询的对应模板的model\_id即可。

#### 二.代码实现

统计按钮:

```text
   $('#btn-count').click(function() {
                layer.open({
                    type: 1,
                    skin: 'layui-layer-rim', //加上边框
                    area: ['900px', '500px'], //宽高
                    content: $('#set-count'),
                    title: "统计数据"
                });
            });
```

统计弹出层:

```text
 <!--统计弹出层-->
    <div id="set-count" style="display:none; width:900px; padding:20px;">
        <form class="layui-form">
            <div class="layui-form-item">
                <label class="layui-form-label"></label>
                <div class="layui-input-block">
                    <label class="layui-form-label">模板信息</label>
                    <label class="layui-form-label" name="count_model_id"></label>
                    <label class="layui-form-label" name="count_model_name"></label>

                    <!-- <input type="hidden" name="model_id" id="model_id"> -->
                    <hr class="layui-bg-red">
                </div>
                <div class="layui-input-block">
                    <label class="layui-form-label">题目信息</label>
                </div>
                <br />


                <div class="layui-input-block">
                    <label class="layui-form-label-col" name="qsn_count"></label>

                </div>
            </div>

        </form>

        <!-- <div class="layui-collapse" name="qsn_count" lay-accordion> </div> -->
    </div>
```

以及统计按钮的ajax刷新：

```text
else if (layEvent == 'count') {
                    layer.msg("加载中", {
                        icon: 16,
                        shade: 0.01,
                        time: '9999999'
                    });
                    $('#set-count label[name="count_model_id"]').text(data.model_id);
                    $('#set-count label[name="count_model_name"]').css("width", "200px");
                  $('#set-count label[name="count_model_id"]').css("width", "200px");
                    $('#set-count label[name="count_model_name"]').text(data.name); //弹出层的初始化
                    var model_id = data.model_id; //将当前选中行的code给code变量
                    var url = "{:url('survey/count')}";
                    var data = {
                        model_id: model_id
                    };
                    $.post(url, data, function(data) {
                        layer.close(layer.index); //关闭正在加载中弹出层

                        if (data.code == 1) {
                            layer.open({
                                type: 1,
                                skin: 'layui-layer-rim', //加上边框
                                area: ['1000px', '600px'], //宽高
                                content: $('#set-count')
                            });
                            $('#set-count label[name="qsn_count"]').html(""); //清空info内容

                            for (var i = 0; i < data.data.length; i++) {
                                var all;
                                var content = " ";
                                var title = '';
                                // alert(data.data[i].content);

                                if (data.data[i].qsn_type == 0 || data.data[i].qsn_type == 1) {
                                    title = '<div class="layui-input-block">第' + (i + 1) + '题:&nbsp;&nbsp;&nbsp;&nbsp;' + data.data[i].content + '</div>';
                                    var count = data.data[i].option_list.length;
                                    for (var j = 0; j < count; j++) {
                                        content = content + '<div class="layui-input-block">' + letter[data.data[i].option_list[j].order_num - 1] + ':' + data.data[i].option_list[j].option_num + '  <br/> 被' + data.data[i].option_list[j].count + '次点击</div><br/>';
                                    }
                                    all = title + content;
                                    $('#set-count label[name="qsn_count"]').append(all);
                                    $('#set-count label[name="qsn_count"]').append(' <hr class="layui-bg-red">');
                                    // else {
                                    title = null;
                                    content = null;
                                    all = null;

                                } else if (data.data[i].qsn_type == 2) {
                                    // alert(data.data[i].content);
                                    title = '<div class="layui-input-block">第' + (i + 1) + '题:&nbsp;&nbsp;&nbsp;&nbsp;' + data.data[i].content + '</div>';
                                    var count = data.data[i].answer.length;
                                    for (var j = 0; j < count; j++) {
                                        content = content + '<div class="layui-input-block">乘客答案:' + data.data[i].answer[j].choose + '</div>';
                                    }
                                    all = title + content;
                                    $('#set-count label[name="qsn_count"]').append(all);
                                    $('#set-count label[name="qsn_count"]').append(' <hr class="layui-bg-red">');
                                    title = null;
                                    content = null;
                                    all = null;
                                }
                                // }
                            };
                        } else {
                            layer.msg(data.msg, {
                                icon: 5
                            });
                        }
                    }, "json");
                }
            });
```

这里我们只上传了一个model\_id作为连表查询的条件。 然后我们看看Controller层代码:

```text
public function count()
    {
        $model_id = input('post.model_id');
        $qsn_list = db('qsn')->where('model_id', $model_id)->field('content,order_num,qsn_id,qsn_type')->select(); //查询到了 这是个二维数组
        $result = [];
        foreach ($qsn_list as $key => $value) {
            // $qsn_list[$key] = db('psg_qsn_r')->where('qsn_id', $value['qsn_id'])->field('count(*) as count,choose')->group('choose')->select();
            if (2 == $value['qsn_type']) {
                $result[$key]['content'] = $value['content'];
                $result[$key]['qsn_type'] =2;
                $ask_detail = db('qsn_detail')->where('qsn_id', $value['qsn_id'])->field('detail_id')->find(); //关联数组
                $answer = db('psg_qsn_r')->where('qsn_id', $value['qsn_id'])->where('detail_id', $ask_detail['detail_id'])->field('choose')->select();
                $result[$key]['answer']=$answer;
                continue;
            }
            $option_list = db('qsn_detail')->where('qsn_id', $value['qsn_id'])->field('option_num,order_num,detail_id')->select(); //小二维数组
            foreach ($option_list as $key1 => $value1) {
                $count = db('psg_qsn_r')->where('qsn_id', $value['qsn_id'])->where('detail_id', $value1['detail_id'])->field('choose')->select();

                $option_list[$key1]['count'] = count($count);

            }
            $result[$key]['content'] = $value['content'];
            $result[$key]['qsn_type'] = $value['qsn_type'];
            $result[$key]['option_list'] = $option_list;
            // $result=array_merge($value, $option_list);
        }
        returnjson([1, 'success', $result]);
    }
```

功能具体的拼接已经说过了，拼二维数组，然后用array\_merge函数合并。

然后我们看看最终效果

#### 三.效果一览

![1.gif](https://upload-images.jianshu.io/upload_images/9003228-bac11a2b0d9d7992.gif?imageMogr2/auto-orient/strip)

OK，完成啦，喜欢这篇文章以及这个系列的小伙伴，就给颗小💗💗吧~

