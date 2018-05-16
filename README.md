# SMS_UMeditor
##SMS中的baidu umeditor定制

富文本是一个相对来说比较坑的东西，这里SMS采用了baidu Umeditor，对其进行了简单的包装，主要是样式上的修改。

### UMeditor

UMeditor，简称UM，是 ueditor 的简版。是为满足广大门户网站对于简单发帖框和回复框的需求，专门定制的在线富文本编辑器，它是ueditor的简易版本，更加轻量，而且不需要插入iframe，采用的是**contenteditable="true"**来使得文档可编辑，非常符合SMS的需求
````html
<div id="editor" class=" edui-body-container" contenteditable="true" style="width: auto; min-height: 1px; z-index: 999; height: auto;"><p>我是一段正文 常规字体/28px(后台编辑14px)/字色:#333333 内容我是一段正文内容我是一段正文内容我是一段正文内容我<span style="font-size: 0.96rem;">是一段正文内容我是一段正文内容我是一段正文内容我是一段正文内容我是一段正文内容我是一段正文内容我是一段正</span>文内容我是一段正文内容我是一段正文内容我是一段正文内容我是一段正文内容我是一段正文内容我是一段正文内容我是一段正文内容</p></div>
````

下面是 官方的文档，可以参考下：
- [UMeditor的文档地址](http://fex.baidu.com/ueditor/)
- [UMeditor的github地址](https://github.com/fex-team/umeditor)


### 引入UMeditor

在SMS中，我们在editor目录下新建一个html2（旧版富文本编辑器是html目录），然后在目录中引入UMeditor的库，主要就是umeditor.js和umeditor.config.js, umeditor.js是总体的编辑器代码，umeditor.config.js是编辑器相关的配置代码。
这个目录里面还有index.js, 是我添加的，主要就是编辑器的入口文件，在VDAPI中调用 **open** 来唤起富文本编辑器时，就会执行index.js的open方法，这个方法主要做的是以下几步：
- 唤起富文本编辑器，并且设置工具栏选项
````js
var ue = UM.getEditor('editor',{
    // 去掉 fontfamily
    toolbar:['undo redo | fontsize | bold italic underline strikethrough | forecolor backcolor | removeformat | lineheight letterspacing justifyleft justifycenter justifyright justifyjustify justifyfull  | link simpleUpload | emotion | horizontal'],
});
````
- 设置富文本编辑器浮动样式（因为在SMS场景里，编辑器需要上浮在编辑的文本之上）
- 设置了图片上传控件和颜色选择器事件监听（这个是调用SMS自带的图片上传控件和颜色选择器）

同理，当编辑区域焦点blur的时候，就会在VDAPI中调用 **hide** 来富文本编辑器时，就会执行index.js的hide方法，来destroy掉富文本编辑器

#### umeditor.config.js

umeditor.config.js里有编辑器所有完整的配置项，这里主要修改的地方有三处

- 图片的上传地址修改：
````js
/**
    * 配置项主体。注意，此处所有涉及到路径的配置别遗漏URL变量。
    */
var env = (location.host.search('.dev.')>-1 || location.host.search('.daily.') > -1) ? '.daily' : location.host.search('.pre.') > -1 ? '.pre' : '';

var vap = env===''?'https://media.api.weidian.com/':'https://mediaapi'+env+'.weidian.com/';
var URL = window.UEDITOR_HOME_URL || '/_/editor/html2/';
var upload_url = vap + 'upload/v2/direct?scope=vteam&fileType=image';
````

- 工具栏上的所有的功能按钮和下拉框的配置，但是其实这个也可以在new编辑器的实例时选择自己需要的重新定义
````js
toolbar:[
    'fontfamily fontsize lineheight undo redo bold italic underline strikethrough forecolor backcolor removeformat justifyleft justifycenter justifyright justifyjustify justifyfull lineheight letterspacing link emotion horizontal'
    // 'source | undo redo | bold italic underline strikethrough | superscript subscript | forecolor backcolor | removeformat |',
    // 'insertorderedlist insertunorderedlist | selectall cleardoc paragraph | fontfamily fontsize' ,
    // '| justifyleft justifycenter justifyright justifyfull lineheight |',
    // 'link unlink | emotion image video  | map',
    // '| horizontal print preview fullscreen', 'drafts', 'formula'
]
````

- 另外就是白名单的添加和删除，在编辑过程中可能有一些比较罕见的标签，比如svg标签，如果不添加到白名单，会被删除或者转义掉
````js
whiteList: {
    a:      ['target', 'href', 'title', 'style', 'class', 'id'],
    //... 省略
    svg:    ['style', 'viewbox', 'x', 'y', 'class', 'id', 'width', 'height', 'xmlns'],
    g:      ['path', 'd', 'style', 'fill'],
    path:   ['path', 'd', 'style', 'fill'],
    video:  ['autoplay', 'controls', 'loop', 'preload', 'src', 'height', 'width', 'style', 'class', 'id']
}
````

#### UMeditor.js

这是编辑器的主要代码，这部分代码很长，之前我主要修改了字体、行间距、字间距等功能，现在基本上不需要修改。如果需要修改特定功能的话，建议先看一下github上的UMeditor代码，把它clone下来，在本地修改了再添加到SMS的编辑器代码中。