# ThinkPHP5实现答题管理系统（二）

好的，现在我们来实现删除题目模板的功能，删除功能不难做，主要我们这次实现的是批量删除功能。

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
                    var url = "{:url('survey/del_model')}";
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
    //删除
    public function del_model()
    {
        //我们将选中行的id_array赋给了device_id /a时进行批量操作的 是进行批量删除的
        $model_id = input('post.model_id/a');
        if (empty($model_id)) {
            returnjson([3, 'warning', '']);
        }
        $str_device_id = implode(',', $model_id);//将数组分割为字符串
        $where['model_id'] = array('in', $str_device_id);
        $is_answer = db('psg_qsn_r')->where($where)->select();
        if (!empty($is_answer)) {
            returnjson([2, '抱歉,这份题被人答过了,无法删除', $model_id]);
        }
        $del_res = db('qsn_model')->where($where)->delete();
        if ($del_res) {
            returnjson([1, 'succes', $model_id]);
        } else {
            returnjson([2, 'fail', $model_id]);
        }
    }
```

这里进行说明一下，ThinkPHP对数组上传，要加入/a进行识别，表明为数组，这里对数组进行了非空判断，以及用implode\(\)函数将数组分割为字符串，进行批量查询，如果题目被人答过了，返回回答过，如果没有被答过题，就删除题目。

我们来试验一下。 ![1.gif](https://upload-images.jianshu.io/upload_images/9003228-1628fad03209d674.gif?imageMogr2/auto-orient/strip)

成功啦，下一节我们讲对应某一套题目模板下的题目的增删。

