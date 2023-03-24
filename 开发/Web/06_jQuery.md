# 1 在文档加载后激活函数
```
$(document).ready(function() {
    alert('lance is a good man.');
})

等价于
windows.onload = function() {
    alert('hahahaha');
}

jQuery的效率高于windows.onload
```

# 2 jQuery和Dom 互相转换
```
$(document).ready(function() {
    // jQuery对象
    var $jqDiv = $("#div01").html("lance is a cool man");
    console.log(1);
    console.log($jqDiv);

    // jQuery -> Dom
    var domDiv01 =  $jqDiv[0];
    console.log(2);
    console.log(domDiv01);

    var domDiv02 = $jqDiv.get(0);
    console.log(3);
    console.log(domDiv02);

    // Dom -> jQuery
    $jqDiv01 = $(domDiv01);
    console.log(4);
    console.log($jqDiv01)

})
```

# 3 jQuery选择器
## 3.1 基本选择器
```
// jQuery选择器
console.log("id选择器");
var $jqDiv = $("#div01").html("lance is a cool man");   // document.getElementById

console.log($jqDiv);

console.log("元素选择器");
var $jqDiv02 = $("div").html("元素选择器");     // document.getElementsByTagName
console.log($jqDiv02);

console.log("类名选择器");
var $jqDiv03 = $(".app");  // document.getElementsByClassName
console.log($jqDiv03);

console.log("复合选择器");
var $jqDiv04 = $(".app, p");    //将每个选择器匹配的元素一起返回
console.log($jqDiv04);

console.log("通配符选择器");
var $jqDiv05 = $("*");  //匹配所有元素
console.log($jqDiv05)
```

## 3.2 层级选择器
```
<div id="div01" class="app"></div>
<div id="div02" class="app"></div>
<div id="div03">
    <p>username</p>
    <div id="div04">
        <p>password</p>
    </div>
</div>
```
```
console.log("祖先后代选择器");
var $jqObj = $("#div03 p").html("祖先后代选择器");
console.log($jqObj);

console.log("父子选择器");
var $jqObj = $("#div03 > p").html("父子选择器");
console.log($jqObj);

console.log("next选择器");
var $jqObj = $("#div01 + div").html("next选择器");
console.log($jqObj);

console.log("sublings选择器");
var $jqObj = $("#div01 ~ div").html("sublings选择器");
console.log($jqObj);

```

## 3.3 过滤选择器
### 3.3.1 简单过滤器
```

```

## 4 选择器
| 选择器 | 实例 | 选取 |
| :----- | :--- | :--- |
* | $("*") | 所有元素
#id | $("#lastname") | id="lastname" 的元素
.class | $(".intro") | 所有 class="intro" 的元素
element | $("p") | 所有 \<p\> 元素
.class.class | $(".intro.demo") | 所有 class="intro" 且 class="demo" 的元素
 | |
:first | $("p:first") | 第一个 \<p\> 元素
:last | $("p:last") | 最后一个 \<p\> 元素
:even | $("tr:even") | 所有偶数 \<tr\> 元素，从0开始
:odd | $("tr:odd") | 所有奇数 \<tr\> 元素
 | | 
:eq(index) | $("ul li:eq(3)") | 列表中的第四个元素（index 从 0 开始）
:gt(no) | $("ul li:gt(3)") | 列出 index 大于 3 的元素
:lt(no) | $("ul li:lt(3)") | 列出 index 小于 3 的元素
:not(selector) | $("input:not(:empty)") | 所有不为空的 input 元素
 | | 
:header | $(":header") | 所有标题元素 \<h1\> \- \<h6\>
:animated |  | 所有动画元素
 | | 
:contains(text) | $(":contains('W3School')") | 包含指定字符串的所有元素
:empty | $(":empty") | 无子（元素）节点的所有元素
:hidden | $("p:hidden") | 所有隐藏的 \<p\> 元素
:visible | $("table:visible") | 所有可见的表格
 | | 
s1,s2,s3 | $("th,td,.intro") | 所有带有匹配选择的元素
 | | 
[attribute] | $("[href]") | 所有带有 href 属性的元素
[attribute=value] | $("[href='#']") | 所有 href 属性的值等于 "#" 的元素
[attribute!=value] | $("[href!='#']") | 所有 href 属性的值不等于 "#" 的元素
[attribute$=value] | $("[href$='.jpg']") | 所有 href 属性的值包含以 ".jpg" 结尾的元素
 | | 
:input | $(":input") | 所有 \<input\> 元素
:text | $(":text") | 所有 type="text" 的 \<input\> 元素
:password | $(":password") | 所有 type="password" 的 \<input\> 元素
:radio | $(":radio") | 所有 type="radio" 的 \<input\> 元素
:checkbox | $(":checkbox") | 所有 type="checkbox" 的 \<input\> 元素
:submit | $(":submit") | 所有 type="submit" 的 \<input\> 元素
:reset | $(":reset") | 所有 type="reset" 的 \<input\> 元素
:button | $(":button") | 所有 type="button" 的 \<input\> 元素
:image | $(":image") | 所有 type="image" 的 \<input\> 元素
:file | $(":file") | 所有 type="file" 的 \<input\> 元素
 | | 
:enabled | $(":enabled") | 所有激活的 input 元素
:disabled | $(":disabled") | 所有禁用的 input 元素
:selected | $(":selected") | 所有被选取的 input 元素
:checked | $(":checked") | 所有被选中的 input 元素