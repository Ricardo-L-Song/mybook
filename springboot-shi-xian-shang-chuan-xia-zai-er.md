# SpringBoot实现上传下载\(二\)

这篇写下载。

### 1.实现思路

上一篇的数据库设计中，我们有一个字段始终没有用到fileName,这是用来给Layer对象存储文件名的,以此来完成文件与对象的对应， ![image.png](https://upload-images.jianshu.io/upload_images/9003228-5aaf0851b2d90a1a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

预览: ![image.png](https://upload-images.jianshu.io/upload_images/9003228-fea3b7d327d48fc1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

### 2.Code

View层: 首先是加载数据表格异步的时候 我们就获取到了fileName，然后通过获取当前行，来获取当前的fileName文件名。

```text
table.on('tool(test)', function(obj) { 

            var data = obj.data; //获得当前行数据
            var layEvent = obj.event; //获得 lay-event 对应的值（也可以是表头的 event 参数对应的值）
            var tr = obj.tr; //获得当前行 tr 的DOM对象
            $ = layui.jquery;
if (layEvent === 'download') { //删除

                var fileName = data.fileName;
             window.location="layer/download?fileName="+fileName;

            }
 });
```

然后不论是上传下载使用ajax都不推荐\(上传下载都属于获取资源请求，a标签等即可实现，而ajax是js异步的封装请求，两者实现目标不一样\)

这里使用window.location来实现对文件的请求。

然后是Controller层:

```text
 //下载
    @RequestMapping(value = "Index/layer/download")
    @ResponseBody
    public Map<String,Object> downloadOne(HttpServletRequest req,HttpServletResponse response) throws IOException{
        String fileName = req.getParameter("fileName");// 设置文件名，根据业务需要替换成要下载的文件名
//        String layerId = req.getParameter("layerId");
        String downloadDir = req.getSession().getServletContext().getRealPath("/") +"upload/";
//        String savePath = req.getSession().getServletContext().getRealPath("/") +"download/";
        downloadDir=downloadDir.substring(0,downloadDir.length()-1);
        downloadDir=downloadDir+"\\";//下载目录
        String realPath=downloadDir+fileName;//
        File file = new File(realPath);//下载目录加文件名拼接成realpath
        if(file.exists()){ //判断文件父目录是否存在
//            response.setContentType("application/force-download");
            response.setHeader("Content-Disposition", "attachment;fileName=" + fileName);

            byte[] buffer = new byte[1024];
            FileInputStream fis = null; //文件输入流
            BufferedInputStream bis = null;

            OutputStream os = null; //输出流
            try {
                os = response.getOutputStream();
                fis = new FileInputStream(file);
                bis = new BufferedInputStream(fis);
                int i = bis.read(buffer);
                while(i != -1){
                    os.write(buffer);
                    i = bis.read(buffer);
                }

            } catch (Exception e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
            System.out.println("----------file download" + fileName);
            try {
                bis.close();
                fis.close();
            } catch (IOException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }


        return api.returnJson(2,"fail"+realPath+fileName);
    }
```

主要就是调IO流，然后下载罢了，拼一下路径和文件名即可

### 3.效果一览

![&#x6548;&#x679C;.gif](https://upload-images.jianshu.io/upload_images/9003228-c1e0765772310eb5.gif?imageMogr2/auto-orient/strip)

觉得有用就给颗小💗💗吧~

