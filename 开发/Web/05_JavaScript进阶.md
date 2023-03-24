[toc]
## 1 BOM
### 1.1 BOM
```
window.document
    是一个BOM对象，表示的是当前所载入的文档(即页面)，但它的方法和属性同时也属于DOM对象所涵盖的范围。

window.frames
    是当前页面中所有框架的集合

window.navigator
    用于反应浏览器及其功能信息的对象

window.screen
    提供浏览器以外的环境信息

window.location
    href属性        控制浏览器地址栏的内容
    reload()       刷新页面 
    reload(true)   刷新页面，不带缓存
    assign()       加载新的页面
    replace()      加载新的页面(注意：不会再浏览器的历史记录表中留下记录)
    
window.history
    window.history.length获取历史记录的长度
    back()     上一页
    forward()  下一页
    go(num)    //num<0时，跳转到自己后方的第num个记录。num>0时，跳转自己前方的第num个记录
```

### 1.2 Window中的方法
```
window.open("yellowWindow.html", "blank", "width=200px, height=400px, top=0px, left=0px");
    width       数值    新窗口的宽度，不能小于100
    height      数值    新窗口的高度，不能小于100
    top         数值    距离屏幕上方的像素
    left        数值    距离屏幕左边的像素
    toolbar     yes no  是否显示工具栏，IE有效
    location    yes no  是否显示地址栏，IE有效
    fullscreen  yes no  全屏显示
```

### 1.3 window中的常用事件
```
    onload onunload     # 加载事件
            window.onload = function() { ... };
                    当页面加载完成的时候会触发该事件；
            window.onunload = function() { ... };
                    当页面完全卸载再触发，只有IE支持；
    
    onscroll    # 滚动事件                
            获取滚动高度
                var scrollTop = document.documentElement.scrollTop||document.body.scrollTop;
    
    onresize    # 窗口变化事件
            window.onresize = function() { ... };
                    当浏览器发生缩放的时候就会反复触发resize事件；
            获取窗口大小
                    var w = document.documentElement.clientWidth || document.body.clientWidth || window.innerWidth;
    				var h = document.documentElement.clientHeight || document.body.clientHeight || window.innerHeight;
    				console.log(document.documentElement.clientWidth, document.body.clientWidth, window.innerWidth, w);
    				console.log(document.documentElement.clientHeight, document.body.clientHeight, window.innerHeight, h);
```

### 1.4 定时器 
```
间歇定时器
        var timer = setInterval(函数名,时间)    // 创建一个间歇性定时器，每间隔 参数2毫秒 执行一次参数1函数；
        window.clearInterval(timer)     // 停止定时器；定时器停止后，无法恢复，只能重新创建；
        
演示定时器
        var timer = setTimeout(函数名，时间)    // 参数2毫秒 以后再执行参数1函数
        window.clearTimeout(timer)      // 关闭定时器
```

## 2 DOM
### 2.1 DOM
```
DOM：文档对象模型(Document Object Model)
        DOM是访问HTML和操作HTML的标准。
        
        Core DOM – 核心DOM 针对任何结构化文档的标准模型
        XML DOM  - 针对 XML 文档的标准模型
        HTML DOM - 针对 HTML 文档的标准模型
        
DOM节点的分类
        1、文档节点         整个HTML文件内容
        2、标签(元素)节点   HTML标签
        3、属性节点         HTML标签的属性
        4、文本节点         标签内的文本
        5、注释节点
        
DOM节点层级关系(DOM树)
    1、父节点(parent node)：父节点拥有任意数量的子节点
    2、子节点(child node)：子节点只能拥有一个父节点
    3、兄弟节点(sibling node)：拥有相同父节点的同级节点
    4、根节点(root node)：一个HTML文档一般只有一个根节点，根节点没有父亲节点，是最上层的节点。
    5、祖先节点：包含子节点的节点都可以叫做祖先节点，其中包括了父节点。
    6、后代节点：一个节点内包含的所有节点，叫做后代节点，其中包括了子节点。
```

### 2.2 获取节点
```
获取元素节点
    通过元素id获取
        var jsDiv = document.getElementById("idDiv")        返回节点对象
        
    通过class属性获取
        var jsDivClass = document.getElementsByClassName("classDiv")    返回节点列表
        
    通过name属性获取
        var jsDivName = document.getElementsByName("inputText")         返回节点列表
            
获取属性节点
    获取    通过 元素节点.属性名 获取       
    修改    元素节点.属性名 = newValue
    
    获取    通过 元素节点.getAttribute("属性名")    可以获取到自定义属性
    修改    元素节点.setAttribute("属性名", "新的属性值")
    
    移除属性，某些低版本浏览器不支持
            元素节点.removeAttribute("属性名")

获取文本节点
    元素节点.innerHTML      从对象的开始标签到结束标签的全部内容,不包括本身Html标签
    元素节点.outerHTML      除了包含innerHTML的全部内容外, 还包含对象标签本身
    元素节点.innerText      从对象的开始标签到结束标签的全部的文本内容
```

#### 2.2.1 行间样式表的读写
```
获取元素节点
	    var jsDiv = document.getElementById("box");

获取style属性节点
	    var jsDivStyle = jsDiv.style;
	
获取样式表中样式属性的属性值
	    style属性节点.样式属性名  
	    元素节点.style.样式属性名   
	    元素节点.style["样式属性名"]
```
####  2.2.2 内部样式表和外部样式表的读写
```
<body>
	<div id="box1"></div>
	<div id="box2"></div>

	<script type="text/javascript">
		/**获取
		 *
		 * IE中：元素节点.currentStyle.样式属性名
		 *      元素节点.currentStyle["样式属性名"]
		 * 其他：window.getComputedStyle(元素节点, 伪类).样式属性名
		 *      window.getComputedStyle(元素节点, 伪类)["样式属性名"]
		 * 
		 * 伪类一般写null即可
		 * 
		 */
		
		//获取元素节点
		var jsDiv1 = document.getElementById("box1");
		var jsDiv2 = document.getElementById("box2");


		var w1 = 0;
		//判断是否是IE浏览器
		if (jsDiv1.currentStyle) {
			//IE浏览器获取样式属性值的方式
			w1 = jsDiv1.currentStyle.width;
		} else {
			//其他浏览器获取样式属性值的方式
			w1 = window.getComputedStyle(jsDiv1, null).width;
		}
		console.log(w1);

		var w2 = 0;
		if (jsDiv2.currentStyle) {
			w2 = jsDiv2.currentStyle.width;
		} else {
			w2 = window.getComputedStyle(jsDiv2, null)["width"];
		}
		console.log(w2);



		//设置样式中的属性的值
		//元素节点.style.样式属性名 = 样式属性值
		jsDiv1.style.backgroundColor = "black";
		jsDiv2.style.backgroundColor = "blue";

	</script>
</body>
```

### 2.3 节点属性
```
        节点类型    nodeName    nodeType    nodeValue
        元素节点    元素名称    1           null
        属性节点    属性名称    2           属性值
        文本节点    #text       3           文本内容不包括HTML
        注释节点    #comment    8           注释内容


节点共有属性
        nodeName、nodeType、nodeValue
    
    
节点层次关系属性
        1、获取当前元素节点的所有的子节点
        		var childNodesArray = jsDiv1.childNodes;
        		
        2、获取当前元素节点的第一个子节点
        		var firstChildNode = jsDiv1.firstChild;
        		
		3、获取当前节点的最后一个子节点
        		var lastChildNode = jsDiv1.lastChild;
        		
		4、获取该节点的文档的根节点，相当于document
        		var rootNode = jsDiv1.ownerDocument;
        		
		5、获取当前节点的父节点
        		var parentNode = jsDiv1.parentNode;
        		
		6、获取当前节点的前一个同级节点
        		var previousNode = jsDiv2.previousSibling;
        		
		7、获取当前节点的后一个同级节点
        		var nextNode = jsDiv2.nextSibling;
        		
		8、获取当前节点的所有的属性节点
        		var allAttributesArray = jsInput.attributes;

```

### 2.4 DOM节点动态操作
```
		//创建元素节点
		var jsDiv1 = document.createElement("div");
		jsDiv1.id = "box1";
		console.log(jsDiv1);


		//父节点.appendChild(子节点);
		//方法将一个新节点添加到某个节点的子节点列表的末尾上
		document.body.appendChild(jsDiv1);


		//父节点.insertBefore(新节点, 子节点)
		//将新节点添加到父节点的某个子节点的前面
		var jsP = document.createElement("p");
		jsP.innerHTML = "关注我";

		var jsD = document.getElementById("box");
		jsD.insertBefore(jsP, jsD.childNodes[3]);
		console.log(jsD);


		//创建文本节点
		var jsStr = document.createTextNode("什么");

		//添加文本节点
		var js2P = jsD.childNodes[1];
		js2P.appendChild(jsStr); 
		

		//替换节点
		//父节点.replaceChild(新节点, 子节点)
		//将父节点中的某个子节点替换成新节点
		var jsDiv2 = document.createElement("div");
		jsDiv2.id = "box2";
		jsDiv1.parentNode.replaceChild(jsDiv2, jsDiv1);
		
		
		//复制节点
		var jsD1 = jsD.cloneNode();//只复制本身
		console.log(jsD1);
		var jsD2 = jsD.cloneNode(true);//复制本身和子节点
		console.log(jsD2);


		alert("删除节点");
		//删除节点
		//父节点.removeChild(子节点)
		//删除父节点下的对应子节点
		jsDiv2.parentNode.removeChild(jsDiv2);


		//offsetParent（参照物父元素）
		var temp = jsD.childNodes[1].offsetParent;
		console.log(temp);
		//当某个元素的父元素或以上元素都未进行CSS定位时，则返回body元素，也就是说元素的偏移量（offsetTop、offsetLeft）等属性是以body为参照物的
		//当某个元素的父元素进行了CSS定位时（absolute或者relative），则返回父元素，也就是说元素的偏移量是以父元素为参照物的
		//当某个元素及其父元素都进行CSS定位时，则返回距离最近的使用了CSS定位的元素
```