[toc]
# 1 
## 1.1 注释标签
```
    <!-- 注释标签 -->
```

## 1.2 换行标签
```
    百度一下<br/>你就知道<br/>
```
## 1.3 水平线标签
```
    这是<hr/>水平线<br/>
```

## 1.4 head标签
```
    <head>
        <title>...</title>
        <meta>
        <link>
        <style>...</style>
        <script>...</script>
    </head>
    
    <head> 标签用于定义文档的头部，它是所有头部元素的容器；
    头部元素有<title>、<script>、 <style>、<link>、 <meta>等标签；
    
    <title>标签
        在<title>和</title>标签之间的文字内容是网页的标题信息，它会出现在浏览器的标题栏中；
        网页的title标签用于告诉用户和搜索引擎这个网页的主要内容是什么，搜索引擎可以通过网页标题，迅速的判断出网页的主题；
        每个网页的内容都是不同的，每个网页都应该有一个独一无二的title；
```



# 2 body标签
```
在<body>和</body>标签之间的内容是网页的主要内容;；
如<h1>、<p>、<a>、<img>等网页内容标签，在这里的标签中的内容会在浏览器中显示出来；
```

## 2.1 段落标签
```
    <p>这是一段文字</p>
```
## 2.2 标题标签
```
    <h1>一级标题</h1>
    <h2>二级标题</h2>
    <h3>三级标题</h3>
    <h4>四级标题</h4>
    <h5>五级标题</h5>
    <h6>六级标题</h6>
```

## 2.3 文本标签
```
    <font size="6" color="red">这是一段文本</font>
        size    字号
        color   颜色
```

## 2.4 字体加粗
```
    <strong>字体加粗</strong>
    <b>字体加粗</b>
```

## 2.5 字体倾
```
    <em>字体倾斜</em>
    <i>字体倾斜</i>
```

## 2.6 删除
```
    <del>删除线</del>
    <s>删除线</s>
```

## 2.7 下划线
```
    <ins>下划线</ins>
    <u>下划线</u>
```


# 3
## 3.1 图片
```
    <img src="mi6.jpg" width='200' height='100' alt="小米6" title='小米6'>
        src     图片来源(必写属性)；
        alt     替换文本(当图片不显示时，显示该文本)；
        title   提示文本(鼠标放到图片上显示的文字)；
        width   图片宽度；
        hight   图片高度；
        注意：当不定义图片宽高时，默认显示原始尺寸；单独指定宽或高时，图片等比例缩放；
```

## 3.2 超链接 
```
    <a href="xiaoai.html" title='小爱音响' target='_blank'>3.2 小爱</a>
        herf    跳转的界面(必写属性)；
        title   提示文本(鼠标放到图片上显示的文字)；
        target  _self   默认值，在当前页面打开新的页面；
                _blank  在新的页面打开新的页面；
```

## 3.3 锚链接
```
    <!-- 先定义一个锚点位置 -->
    <!-- <p id='top'> -->
    <!-- 再定义锚点的超链接 -->
    <a href="#top">回到顶部</a>
```

## 3.4 空链
```
    <a href="#">这是一个空链</a>
        空链会指向自己
```

## 3.5 压缩文件下载(不推荐)
```
    <a href="03.zip">不推荐使用的压缩包</a>
```

## 3.6 超链接优化写法
```
    <!-- <base target='_blank'> -->
    <!-- 这个写在title中，设置全局新页面打开方式 -->
    <!-- 该设置的优先级低于超链接的target参数 -->
```
# 4 特殊字符
|  特殊符号   | 字符实体 |
| :---------: | :------: |
|    空格     | \&nbsp;  |
|  大于号(>)  |  \&gt;   |
|  小于号(<)  |  \&lt;   |
|   引号(")   | \&quot;  |
| 版权符号(@) | \&copy;  |


# 5
## 5.1 无序列表
```
    <ul type='square'>
        <li>小米6</li>
        <li>小米note3</li>
        <li>红米
            <ul>
                <li>红米5s</li>
                <li>红米note5</li>
            </ul>
        </li>
    </ul>
    <br>
        type    square    小方块
                disc      实心小圆圈
                circle    空心小圆圈
```

## 5.2 有序列表
```
    <ol type='i' start='3'>
        <li>MIX</li>
        <li>MIX2</li>
        <li>MIX2S</li>
    </ol>
    <br>
        type    1    阿拉伯数字序号(默认)
                a    小写英文字母序号
                A    大写英文字母序号
                i    小写罗马数字序号
                I    大写罗马数字序号
        start   3    起始的序号，只能写阿拉伯数字
```

## 5.3 自定义列表
```
    <dl>
        <dt>游戏本</dt>          <!-- 小标题 -->
        <dd>dell游匣</dd>        <!-- 解释标题 -->
        <dd>lenovoR720</dd>      <!-- 解释标题 -->
        <dd>神舟战神</dd>
    </dl>
```

## 5.4 音乐标签
```
    <embed src="周杰伦 - 告白气球.mp3" autostart="false" hidden="false"></embed>
        <br>
        src         wav、mp3文件路径(必写属性)；
        autostart   true,false  默认true
        hidden      true(隐藏播放界面)
```

# 6 表格
```
语法：
    <table>
        <tr>
            <th>姓名</th><th>年龄</th><th>班级</th>
        </tr>
        <tr>
            <td>Lance</td><td>18</td><td>Python</td>
        </tr>
    </table>
    
对齐：
        align:
                left    左对齐
                center  居中对齐
                right   右对齐
        valign：
                top         顶端对齐
                middle      居中对齐
                bottom      底端对齐
                baseline    基线对齐

跨列：
        <td colspan="2">18</td>

跨行：
        <td rowspan="2">18</td>
```
## 6.1 表格标签
```
（1）<table>：定义表格，表格的边界标签，必定包裹表格的其他元素标签
（2）<tr>：定义表格的行
（3）<th>：定义表格的表头，需要被<tr>包裹
（4）<td>：定义表格的单元，需要被<tr>包裹
（5）<thead>：定义表格的页眉，表格分组标签，可将表格分割
（6）<tbody>：定义表格的主体，表格分组标签，可将表格分割
（7）<tfoot>：定义表格的页脚，表格分组标签，可将表格分割
        如果使用 thead、tfoot 以及 tbody 元素，您就必须使用全部的元素。
        它们的出现次序是：thead、tfoot、tbody，这样浏览器就可以在收到所有数据前呈现页脚了
（8）<caption>：定义表格标题
```

## 6.2 表格属性
```
（1）colspan=“value”        合并列
（2）rowspan=“value”		合并行
（3）align=“left/center/right”      水平对齐
（4）valign=“top/bottom/middle/baseline”	垂直对齐方式
（5）cellpadding=“value”	单元边沿与其内容之间的空白
（6）cellspacing=“value”	单元格之间的空白
```

## 6.3 表格的CSS属性
```
caption-side:top/bottom/left/right ;    设置表格标题放置的位置。
说明：
        left，right位置只有火狐识别；
        top，bottom IE7上版本支持；
        IE7以下版本只识别top，不支持其他；
        
border-spacing:value;   单元格间距
说明：
        该属性必须给table添加
        
border-collapse:separate(边框分开)/collapse(边框合并)

empty-cells:show/hide; 无内容单元格显示、隐藏

table-layout:auto/fixed; 
说明：
        自动表格布局：
                列的宽度是由列单元格中没有折行的最宽的内容设定的。
                缺点：较慢（需要在确定最终的布局前访问表格中的所有内容）
        固定表格布局：加快运行的速度，允许浏览器更快的对表格进行布局。缺点：不太灵活。
```

# 7 表单

## 7.1 表单域
```
语法：
        <form [属性名称=“值”]>
        </form>

常用的属性：
        name=“值”：     规定表单的名称
        action=“值”：   提交表单URL
        method=“get/post”： 提交方式
        enctype=“值”：  规定在发送表单数据之前进行编码
                可能的值：
                        ”application/x-www-form-urlencoded”
                        ”multipart/form-data”
                        ”text/plain”
        target=“_black/_self/_parent/_top”：    何处打开表单URL

H5新增属性：
        autocomplete=“on/off”：是否启动表单自动完成
        novalidate=“novalidate”：不验证表单

示例：
    <form name=“xxx”method=“get” action=“yyy” enctype=“multipart/form-data”>
    </form>

```

## 2.2 表单控件
```
（1）文本框
        <input type=“text” [value=“默认值”]/>
        
（2）密码框
        <input type=“password” [placeholder=“密码”] />
        
（3）提交按钮
        <input type=“submit” value=“按钮内容”/>
        
（4）重置按钮
        <input type=“reset” value=“按钮内容”/>
        
（5）单选框/单选按钮
        <input type=“radio” name=“xxx”[checked=“checked”]/>
        
（6）按钮
        <input type=“button” value=“按钮内容”/>
        
（7）复选框
                <input type=“checkbox” name=“xxx”[disabled=“disabled”] />
                
（8）下拉菜单
        <select name=“xxx”>
        	<option>菜单内容</option>
        </select>
        
（9）多行文本框
        <textarea name=“xxx” cols=“字符宽度”rows=“行数”> </textarea>

（10）表单字段集
        <fieldset></fieldset>
        功能：
                相当于一个方框，在字段集中可以包含文本和其他元素。
                该元素用于对表单中的元素进行分组并在文档中区别标出文本。
                fieldset元素可以嵌套，在其内部可以在设置多个fieldset对象。
                
（11）字段级标题
        <legend></legend>
        功能：
        legend元素可以在fieldset对象绘制的方框内插入一个标题。
        legend元素必须是fieldset内的第一个元素。
        
（12）表单元素
        上传文件框（文件域）
                <input type=“file”/>
                上传多个文件，增加 multiple 属性；
        图像域
                <input type=“image” width=“100” height=“100”border=“2”src=“上传图片”/>
        提示信息标签
                <lable for=“绑定控件id名”></lable>
                功能：
                        label 元素用来定义标签，为页面上的其他元素指定提示信息。
                        要将 label 元素绑定到其他的控件上，可以将 label 元素的 for 属性设置为与该控件的 id 属性值相同。
                        label 元素不会向用户呈现任何特殊效果。不过，它为鼠标用户改进了可用性。
                        如果在 label 元素内点击文本，就会触发此控件。
                        就是说，当用户选择该标签时，浏览器就会自动将焦点转到和标签相关的表单控件上。

```

## 
```

```

## 
```

```