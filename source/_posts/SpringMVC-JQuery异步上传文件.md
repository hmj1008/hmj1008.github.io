---
title: SpringMVC+JQuery异步上传文件
date: 2017-04-18 17:17:19
toc: true
tags:
    - 代码
    - spring
    - jquery
---
> 
关于文件上传在许多项目中都是少不了的，javaWeb开发尤其以spring流行。这里扣出项目代码方便搬砖。另外，关于富媒体编辑同样在项目中或多或少遇到，在这里也顺带把wysihtml5这个东东坑我一中午的坑放一下。
<!--more-->

## SpringMVC+JQuery异步上传文件
假定项目能正常跑通非文件处理的Controller。
### 1、spring-web.xml追加如下配置：
```xml
<!-- 上传文件拦截，设置最大上传文件大小   10M=10*1024*1024(B)=10485760 bytes -->  
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">  
    <property name="maxUploadSize" value="10485760" />  
</bean>
```
### 2、UploadController代码（接收Ajax请求）
```java
// 上传控制器
@Controller 
@RequestMapping("/upload")
public class UploadController {
    // 处理图片文件上传 指定请求方式POST
    @RequestMapping(value = "/imageUpload", method = RequestMethod.POST)
    public void imageUpload(HttpServletRequest request,HttpServletResponse response,HttpSession session) {
        // Ajax 请求除文件外的一个参数，可多个。这里key=filePart
        String filePart = request.getParameter("filePart");
    	// 获取请求里的文件数据
	    MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;
	    Map<String, MultipartFile> fileMap = multipartRequest.getFileMap();    
        MultipartFile mf = null;
	    for (Map.Entry<String, MultipartFile> entity : fileMap.entrySet()) {   
    	    // 获取需上传文件  这里假定只有一个文件。
            // 多个暂时不讨论 ，不过套路也是一样，只是前端ajax套路不同
        	mf = entity.getValue();  
    	}
    	// 文件检查，首先确定文件不为空
        if (!mf.isEmpty()) {
            //当文件超过设置的大小时，则不运行上传
            if (mf.getSize() > fileSize) {
                backInfo(response, false, 2, "文件超大！");
                return;
            }
            // 获取原文件名 
        	String fileName = mf.getOriginalFilename();
        	// 获取文件后缀名
        	String fileSuffix = fileName.substring(mf.getOriginalFilename().lastIndexOf(".")+1);
            // 这里获取当前项目作为保存文件的目录
            String curProjectPath = session.getServletContext().getRealPath("/");  
            String saveDirectoryPath = curProjectPath + "/" + "uploadFiles";  
            File uploadDir = new File(saveDirectoryPath);
            
            try {
                //创建文件uploadDir: 目录, newname: 文件在目录里保存的新文件名
                File saveFile = new File(uploadDir, newname);
                //保存文件
                mf.transferTo(saveFile);
                // 返回前端上传成功后图片的地址...
            } catch (Exception e) {
            	logger.error(e.getMessage(), e);
                backInfo(response, false, 1, "写入文件失败！");
                return;
            }
        } else {
            backInfo(response, false, -1, "文件为空！");
            return;
        }
    }
}
```

### 3、前端Ajax请求
```html
<!-- 这里表示在一个div里面一张用来接受用户触发选择文件的图片img -->
<!-- 另外的一个input 标签隐藏，不显示。只是作为容器装文件 -->
<script src="<%=path%>/resource/vendors/jquery-1.9.1.min.js"></script>
<div class="controls">
 <img  id="gd_img" onclick="click_gd_img_input()"  src="<%=path%>/resource/images/eg.jpg"  height="150"width="150"/>
 <input id="gd_img_input" type="file" style="display:none" >
</div>
```

```javascript
// 触发选择文件
function click_gd_img_input(){
     $("#gd_img_input").click(); 
}

// 选择图片触发#gd_img_input的change事件，上传，预览
$(function() {
    $("#gd_img_input").change(function(){
  	    var file = this.files[0] ? this.files[0] : null;
  	    if (!file) { return false; }			
  	    var formData = new FormData();
        formData.append("file", file);
        formData.append("filePart", "goodsDetail");
        //ajax上传
        $.ajax({
	         url : "<%=path%>/upload/imageUpload",
	         type : "POST",
	         cache : false,
	         enctype:"multipart/form-data",
	         data : formData,
	         processData : false,
	         contentType : false
        }).done(function(result) {
             var data = JSON.parse(result);
             console.log(data);
             //预览  		
             $("#gd_img").attr("src","<%=path%>/uploadFiles/"+data.fileName);
        });
    });
});
```
## wysihtml5 使用
### 1、引入依赖
```html
<script src="<%=path%>/resource/vendors/jquery-1.9.1.min.js"></script>
<script src="<%=path%>/resource/bootstrap/js/bootstrap.min.js"></script>
<link href="<%=path%>/resource/vendors/wysiwyg/bootstrap-wysihtml5.css" rel="stylesheet" media="screen">
<script src="<%=path%>/resource/vendors/wysiwyg/wysihtml5-0.3.0.js"></script>
<script src="<%=path%>/resource/vendors/wysiwyg/bootstrap-wysihtml5.js"></script>

<div class="controls" id="gd_detail_div_editor">
    <textarea id="gd_detail_editor" class="input-xlarge textarea" placeholder="Enter text ..."
     style="width: 810px; height: 200px"></textarea>
</div>
```
### 2、应用
```javascript
$("#gd_detail_div_editor").empty();                 	
$("#gd_detail_div_editor").append("<textarea id='gd_detail_editor' class='input-xlarge textarea' placeholder='Enter text ...' style='width: 810px; height: 400px'></textarea>");
// 初始化数据，假设刚从数据库里查出历史数据，现在做修改操作。
$("#gd_detail_editor").val(data.data.detail_info);
// 实例化
$("#gd_detail_editor").wysihtml5();  
// 获取值，以供ajax提交到服务器保存到数据库
$("#gd_detail_editor").val()
```