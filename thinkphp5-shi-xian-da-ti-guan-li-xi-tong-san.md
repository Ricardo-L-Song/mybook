# ThinkPHP5实现答题管理系统（三）

这是这篇系列的第三篇文章，实现的是对应模板下题目及选项的增删。

#### 一.数据库表设计

![qsn&#x6570;&#x636E;&#x8868;.PNG](https://upload-images.jianshu.io/upload_images/9003228-980062bedd8f2191.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![detail&#x8868;.PNG](https://upload-images.jianshu.io/upload_images/9003228-852100eaacad2b2c.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

一套模板下可能有很多道题目，一对多关系，model\_id对应模板id，qsn\_id为题目的主键，model\_name对应模板名称，order\_num对应模板下题目的排序号，content对应题目的内容，qsn\_type对应题目的类型，这里有三种类型 type0为单选,type1为多选,type2为问答。 一道题也有可能对应很多选项，同理。这里的option\_num对应选项的内容，order\_num对应选项的排序号

#### 二.UI设计及功能设计

![1.PNG](https://upload-images.jianshu.io/upload_images/9003228-99d2a74ae9b66f0c.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![2.PNG](https://upload-images.jianshu.io/upload_images/9003228-89f352d0183cccde.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![3.PNG](https://upload-images.jianshu.io/upload_images/9003228-fdbf5dcce4c9f23f.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![4.PNG](https://upload-images.jianshu.io/upload_images/9003228-800a7a1cb30e1b8b.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

功能:当我们点击上面的数据表格1时，对复选框进行监听显示数据表格2，题目的数据表格，点击添加题目按钮时，我们弹出选项框，填充题目内容，并且选择题目类型且进行选项的增删。

#### 三.代码实现：

由于本篇专注于题目的增删，对数据表格1复选框的监听，可以见layui官方文档。

View层实现: 首先是对1.PNG中添加题目按钮的点击

```text
            //弹出添加题目窗口
            $('#btn-add').click(function() {
                // alert(t_model_id);


                layer.open({
                    type: 1,
                    skin: 'layui-layer-rim', //加上边框
                    area: ['660px', '350px'], //宽高
                    content: $('#set-add-qsn'),
                    cancel: function(index, layero) {
                        $('#input_qus').empty();
                        layer.close(index)
                        k = 0;
                        $('#set-add-qsn input[name="content"]').val("");
                        $('#set-add-qsn div[name="input_qus"]').empty();

                        return false;
                    },

                });

            });
```

即弹出2.PNG的选项框（id为set-add-qsn的选项框）。

```text
    <!--添加题目弹出层-->
    <div id="set-add-qsn" style="display:none; width:550px; padding:20px;">
        <form class="layui-form" lay-filter="qsn_form">
            <div class="layui-form-item">
                <label class="layui-form-label">题目内容</label>
                <div class="layui-input-block">
                    <input type="text" name="content" required lay-verify="required" placeholder="请输入题目内容" autocomplete="off" class="layui-input">
                </div>
            </div>
            <div class="layui-form-item">
                <label class="layui-form-label">请选择问题的类型:</label>
                <div class="layui-input-block">
                    <select name="qsn_type" lay-verify="required" style="width: 212px;">
                      <option value=""></option>
                      <option value="0">单选题</option>
                      <option value="1">多选题</option>
                      <option value="2">问答题</option>
                    </select>
                </div>
                <br/>
                <div class="layui-input-block" id="input_qus" name="input_qus">

                </div>
            </div>

            <div class="layui-form-item">
                <div class="layui-input-block">

                    <button type="button" class="layui-btn" id="add_op">添加选项</button>
                    <button type="button" class="layui-btn" id="del_op">减少选项</button>
                    <button type="button" class="layui-btn" lay-submit lay-filter="formDemo" id="commit">提交</button>
                    <button type="reset" class="layui-btn layui-btn-primary">重置</button>
                </div>
            </div>
        </form>
    </div>
```

content代表题目内容，qsn\_type下拉框代表题目类型，input\_qus是根据qsn\_type来显示的添加按钮，删除按钮的域。 然后我们声明一个letter数组，用来存放26个英文字母。并且声明k，来记录增加选项和删除选项的个数，并且对K值作为letter数组下标的索引

```text
 var k = 0;
            var letter = new Array('A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z');
            $('#add_op').click(function() {
                $('#input_qus').append(
                    '<label class="layui-form-label" id="qs_letter" name="qs_letter">' + letter[k] + '</label>' + '<div class ="layui-input-block" id="qs_detail"><input type = "text" name = "user_op"  lay-verify="required" placeholder = "请输入选项' + (k + 1) + '" autocomplete = "off" class = "layui-input input_qs" id = "user_op' + k + '"></div>'
                );
                k++;
                alert(k);
            });

    $('#del_op').click(function() {

                $('#input_qus input[name="user_op"]:eq(' + (k - 1) + ')').remove(); //input_qs的第k个remove
                $('#input_qus label[name="qs_letter"]:eq(' + (k - 1) + ')').remove(); //input_qs的第k个remove
                $('#input_qus div[id="qs_detail"]:eq(' + (k - 1) + ')').remove();
                if ($('#input_qus :eq(' + (k - 1) + ')' != null)) {
                    k--;
                }
                if ((k + 1) == 0) {
                    layer.msg("无法删减更多选项了");
                    $('#input_qus').empty();
                    k = 0;
                }
                alert(k);
            });
```

这样就能实现对单选\(type=0\)和多选问题\(type=1\)选项的增删。

```text
      form.on('select()', function(data) {
                t_qsn_type = data.value;
                // alert($('#set-add-qsn select[name="op_type"]').val());
                if (data.value == 2) {
                    $('#add_op').attr('disabled', true);
                    $('#add_op').hide();
                    $('#del_op').attr('disabled', true);
                    $('#del_op').hide();
                    $('#input_qus').hide();
                    // form.render('select');
                } else {
                    $('#add_op').attr('disabled', false);
                    $('#add_op').show();
                    $('#del_op').attr('disabled', false);
                    $('#del_op').show();
                    $('#input_qus').show();
                }

            });
```

同时对type=0，1以及type=2（问答题）进行不同按钮功能的UI显示，现在我们就能进行题目的增加啦。 ![&#x9898;&#x76EE;&#x9009;&#x9879;.PNG](https://upload-images.jianshu.io/upload_images/9003228-33d1edc5e09dc7ad.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

```text
 //确认提交问题到后台的按钮
            $('#commit').click(function() { //增加选项
                var a = [];
                $("#input_qus input[name='user_op']").each(function() { //往数组里添加input里获取到的值
                    a.push($(this).val()); //往a数组里添加
                });
                alert(a);
                var content = $('input[name="content"]').val(); //获取值

                if (content !== '') {
                    //打开正在加载中弹出层
                    layer.msg('加载中', {
                        icon: 16,
                        shade: 0.01,
                        time: '9999999'
                    });
                    var qsn_type = t_qsn_type;
                    var model_id = t_model_id; //选择到了模板的id 然后随机生成一个qsnid 这个qsn的id下生成诸多问题选项
                    var model_name = t_model_name;
                    var url = "{:url( 'manager/qs_add_mould')} ";
                    var data = {
                        model_id: model_id, //模板id用来生成qsn_id
                        content: content, //用来存储问题内容
                        model_name: model_name,
                        a: a, //用来存储所有问题的数组
                        qsn_type: qsn_type
                    }
                    $.post(url, data, function(data) { //确认提交
                        k = 0;
                        $('#set-add-qsn input[name="content"]').val("");
                        $('#set-add-qsn div[name="input_qus"]').empty();
                        layer.closeAll();
                        if (data.code == 1) {
                            layer.msg(data.msg, {
                                icon: 6
                            });
                            refresh_table1();
                        } else {
                            layer.msg(data.msg, {
                                icon: 5
                            });
                        }
                    }, "json ");
                }
            });
```

上传五个参数，model\_id和model\_name对应题目的模板表，content对应题目的内容，a用来对应所有选项的数组，qsn\_type对应题目类型，我们接下来，将在后台进行qsn表的增加以及detail选项表的批量增加。

**Controller层。**

```text
 public function qs_add_mould()
    {
        // model_id: model_id, //模板id用来生成qsn_id
        // content: content, //用来存储问题内容
        // a: a //用来存储所有问题的数组
        $data['model_id'] = input('post.model_id');
        $data['content'] = input('post.content');
        $data['qsn_id'] = uniqid('qsn_id', true);
        $data['flight_id'] = uniqid('flight_id', true);
        $data['qsn_type']=(int)input('post.qsn_type');

        $a = input('post.a/a');
        //这里我们要将数组输出为字符串 然后对字符串判断是否为空字符串
       //这里a为数组
        if($data['qsn_type']==2){
            $order_num_list = db('qsn')->where('model_id', $data['model_id'])->order('order_num desc')->field('order_num')->find(); //找到这个order_num
            $order_num = $order_num_list['order_num']; //这里假设取到了值
            $order_num = $order_num + 1;
                $data['order_num'] = $order_num;
                $data['model_name'] = input('post.model_name');
                   //查询题目 是否重复
            $chk_content = db('qsn')->where('content', $data['content'])->find($data);
            if (empty($chk_content)) {
                //插入数据库
                $insert = db('qsn')->insert($data);
                $data_detail['qsn_id']=$data['qsn_id'];
                $data_detail['option_num']=0;
                $data_detail['order_num']=0;
                $data_detail['detail_id']=uniqid('detail_id', true);
                $insert = db('qsn_detail')->insert($data_detail);
                returnjson([1, '添加成功', '']);
            }
            $order_num=null;
        }
        if (empty($a)) {
            returnjson([2, '题目不完全,请填充选项', $data['qsn_type']]);

        } else {
           //用inarray方法判断数组元素中是否含有''空字符串
           if (in_array("", $a)){
            returnjson([2, '题目不完全,请填充选项', $data['qsn_type']]); 
           }

            $order_num_list = db('qsn')->where('model_id', $data['model_id'])->order('order_num desc')->field('order_num')->find(); //找到这个order_num
        $order_num = $order_num_list['order_num']; //这里假设取到了值
        $order_num = $order_num + 1;
            $data['order_num'] = $order_num;
            $data['model_name'] = input('post.model_name');

            //查询题目 是否重复
            $chk_content = db('qsn')->where('content', $data['content'])->find($data);
            if (empty($chk_content)) {
                //插入数据库
                $insert = db('qsn')->insert($data);
                $data1['qsn_id'] = $data['qsn_id'];
                $num = count($a);
                for ($i = 0; $i < $num; ++$i) {
                    $j = $i;
                    $data1['type']=0;
                    $data1['detail_id'] = uniqid('detail_id', true);
                    $data1['option_num'] = $a[$i];
                    $data1['order_num'] = ++$j;
                    $insert = db('qsn_detail')->insert($data1);
                }
                returnjson([1, '添加成功', $a]);
            } else {
                returnjson([3, '该问题已存在', '']);
            }
        }
    }
```

同样对a数组进行字符串分割，然后进行题目表qsn的增加以及qsn\_detail表的批量增加（通过对a数组长度的判别来进行批量增加次数的控制）

然后可以看见已经添加成功了。 ![add\_qsn.PNG](https://upload-images.jianshu.io/upload_images/9003228-db0c169684b1c7de.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

**然后我们对题目作删除。删除同样跟删除模板一样，传qsn\_id数组到后台，删除的时候作级联删除，删除qsn表中的题目 以及qsn\_detail表中的选项，在此之前，判断有没有用户作答过。**

View层ajax实现:

```text
            $("#btn-delete-all").click(function() {
                var model_id = t_model_id;
                if (model_id) {
                    layer.confirm('您确定要删除这些数据吗？', function(index) { //第二个参数为可选参数 点击确认以后的回调函数
                        layer.msg('加载中', {
                            icon: 16,
                            shade: 0.01,
                            time: '9999999'
                        });
                        var url = "{:url('manager/del_option')}";
                        var data = {
                            qsn_id: id_array //将当前选中行的qsn_id传到后端
                        }
                        $.post(url, data, function(data) {


                            layer.close(layer.index); //关闭加载中弹窗
                            if (data.code == 1) {
                                layer.msg(data.msg, {
                                    icon: 6
                                });
                                refresh_table1();
                            } else {
                                layer.msg(data.msg, {
                                    icon: 5
                                });
                            }
                        }, "json"); //一定不要忘了这个json！！！
                    })
                } else {
                    layer.msg("请先选择模板");
                }
            });
```

Controller层逻辑:

```text
  public function del_option()
    {
        $qsn_id = input('post.qsn_id/a');
        if (empty($qsn_id)) {
            returnjson([3, 'warning', '']);
        }
        $str_device_id = implode(',', $qsn_id);
        $where['qsn_id'] = array('in', $str_device_id);
        $is_answer=db('psg_qsn_r')->where($where)->select();
        if(!empty($is_answer)){
            returnjson([2, '抱歉,这份题被人答过了,无法删除', $qsn_id]);
        }
        $del_res = db('qsn')->where($where)->delete();
        $del_res1 = db('qsn_detail')->where($where)->delete();
        if ($del_res) {
            returnjson([1, 'succes', '']);
        } else {
            returnjson([2, 'error', '']);
        }
    }
```

#### 四.效果展示

之前增加的题目数据已经显示在数据表格了 ![add\_qsn.PNG](https://upload-images.jianshu.io/upload_images/9003228-ee4e002b757ebc6c.PNG?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

然后我们删除题目。 ![1.gif](https://upload-images.jianshu.io/upload_images/9003228-8c71a169492164dc.gif?imageMogr2/auto-orient/strip)

OK，完成啦

