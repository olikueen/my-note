[toc]

### 1 CSS盒模型
#### 1.1 盒模型定义
```
盒模型是css布局的基石，它规定了网页元素如何显示以及元素间相互关系。
css定义所有的元素都可以拥有像盒子一样的外形和平面空间，即都包含边框（border)、外边距(margin)、内边距(padding)、内容区(content)。
```

#### 1.2 盒模型模式
```
标准盒模型：
        width 和 height 对应的是content(内容)的宽和和高；

IE盒模型：
        width 和 height 对应的是border(边框)的宽和和高；
```

#### 1.3 盒的内容
```
overflow：
        overflow属性指定如果内容溢出一个元素的框，对应的处理方法；
        
        overflow: visible/hidden/scroll/auto ;
        
                visible：默认值，内容不会被剪裁，会呈现元素框之外；
                hidden：内容被剪裁，其余内容不可见；
                scroll：内容被剪裁，滚动条显示其余内容；
                auto：如果内容被剪裁，则浏览器以滚动条显示其余内容；

        overflow-x  属性指定如果它溢出了元素的内容区是否剪辑左/右边缘内容；
        overflow-y  属性指定如果它溢出了元素的内容区是否剪辑顶部/底部边缘内容；
        

text-overflow：
        text-overflow属性指定当文本溢出包含它的元素，对应的处理方法；
        
        text-overflow: clip/ellipsis/string；
                clip：修剪文本；
                ellipsis：显示省略符号来代表被修剪的文本；
                string：使用给定的字符串来代表被修剪的文本；

注意：
        text-overflow 必须和 overflow:hidden;white-space:nowrap; 一起使用；
        text-overflow 需要注意浏览器的兼容情况，火狐下有显示；
```

#### 1.4 盒模型样式的分类定义与应用
```
（1）盒模型的宽度
        选择器{width:值;}
        
（2）盒模型的高度
        选择器{height:值;}
        
（3）盒模型的背景
       选择器{background-image:url(图片地址);}
       选择器{background-color:值;}
       
（4）盒模型的边框(border)

（5）盒模型的外边距(padding)

（6）盒模型的内边距(margin)
```

##### 1.4.1 边框(border)
```
元素的边框 (border) 是围绕元素内容和内边距的一条或多条线；
CSS border 属性允许规定元素边框的样式、宽度和颜色；
```

| 属性          | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| border        | 简写属性，用于把针对四个边的属性设置在一个声明；             |
| border-style  | 用于设置元素所有边框的样式，或者单独地为各边设置边框样式；   |
| border-width  | 简写属性，用于为元素的所有边框设置宽度，或者单独地为各边边框设置宽度； |
| border-color  | 简写属性，设置元素的所有边框中可见部分的颜色，或为 4 个边分别设置颜色； |
| border-bottom | 简写属性，用于把下边框的所有属性设置到一个声明中；           |
| border-left   | 简写属性，用于把左边框的所有属性设置到一个声明中             |
| border-right  | 简写属性，用于把右边框的所有属性设置到一个声明中             |
| border-top    | 简写属性，用于把上边框的所有属性设置到一个声明中             |


```
注意：
        如果要设置盒子的边框，必须制定边框线型，如果只制定了线型，
        那么颜色和宽度将会采用默认值


border-style:solid\dashed\dotted\double\none
        solid:实线
        dashed:虚线
        dotted:点状线
        double:双线

使用方法：
        边框border:
                线型（solid/dashed/dotted/double）  粗细 (数值+单位) 颜色;
        右边框border-right:
                线形（solid/dashed/dotted/double）  粗细 (数值+单位) 颜色;
        左边框 border-left:
                线形（solid/dashed/dotted/double）  粗细 (数值+单位) 颜色;
        上边框 border-top:
                线形（solid/dashed/dotted/double）  粗细 (数值+单位) 颜色;
        下边框 border-bottom:
                线形（solid/dashed/dotted/double）  粗细 (数值+单位)颜色;
        
        border-top-style/border-top-width/border-top-color--->border-top
        
        border-right-style/border-right-width/border-right-color--->border-right
        
        border-bottom-style/border-bottom-width/border-bottom-color--->border-bottom
        
        border-left-style/border-left-width/border-left-color--->border-left
        
        border-style/border-width/border-color----border
```

##### 1.4.2 内边距(padding)
```
padding：填充，元素边框与元素内容之间的区域，称之为内边距；

用法：
        用来调整内容在容器中的位置关系；
        用来调整子元素在父元素中的位置关系；注：padding属性需要添加在父元素上；
        padding值是额外加在元素原有大小之上的，如想保证元素大小不变，需从元素宽或高上减掉后添加的padding属性值；

属性值四种方式：
          四个值：上    右    下    左  {padding:10px 20px 30px 40px;}
          三个值：上    左右  下        {padding:10px 20px 30px;}
          二个值：上下  左右            {padding:10px 20px;}
          一个值：四个方向              {padding:2px;}

说明：
        可单独设置一方向填充，如：
                上方向 padding-top:10px;
                右方向 pahdding-right:10px;    
                下方向 padding-bottom:10px;
                左方向 padding-left:10px;

```

##### 1.4.3 外边距(margin)
```
margin：盒子与盒子之间的距离

属性：
        margin
        margin-top：上边界
        margin-right：右边界
        margin-bottom：下边界
        margin-left：左边界

属性值四种方式：
        四个值：上   右     下   左 {margin:10px 20px 30px 40px;}
        三个值：上   左右   下      {magin: 10px 20px 30px;}
        二个值：上下 左右           {margin:10px 20px;}
        一个值：四个方向            {margin:2px;}

说明：
        {margin: 0 auto; } 在浏览器中横向居中
        可单独设置一方向边界，如：
                上边界margin-top:10px;
                右边界margin-right:10px;
                下边界margin-bottom:10px;
                左边界margin-left:10px;
```

##### 1.4.4 外边距的合并
```
（1）当两个盒子纵向排列的时候，他们的距离为两个盒子中最大的那个外边距；

（2）当两个盒子横向排列时，他们的距离两个盒子之间的外边距之和；
```

### 2 浮动
#### 2.1 页面布局方式
```
页面布局方式，主要包含：
        文档流（常规流）、浮动流（脱离文档流）。


文档流：
        文档流中元素框的位置由元素在HTML中的位置决定。
        
        块级元素从上到下依次排列，框之间的垂直距离由框的垂直margin计算得到。
        行内元素在一行中水平布置。
        
        文档流就是html文档中的元素如块级元素、行内元素依据他们的显示属性按照在文档中的先后次序依次显示。
        块级元素就占一行或多行，行内元素就和其他元素共处一行，一个萝卜一个坑。


浮动流：
        元素从正常的排列顺序被抽离
        
        浮动可以使元素向左或向右移动，直到它的外边缘碰到包含框或另一个浮动框的边框为止。 
        由于浮动框不在正常文档流中，所以标准文档流中的块框表现得就像浮动框不存在一样。
```

#### 2.2 浮动属性
```
浮动属性：
        float:left（向左浮动）/right（向右浮动）/none(默认值);

float 属性定义元素在哪个方向浮动。
        通过float属性实现元素的浮动。
        以往这个属性总应用于图像，使文本围绕在图像周围，不过在 CSS 中，任何元素都可以浮动。
        浮动元素会生成一个块级框，而不论它本身是何种元素。

如果浮动非替换元素，则要指定一个明确的宽度；否则，它们会尽可能地窄。
```

#### 2.3 注意
1. 浮动元素不在标准文档流中，所以浮动后面紧跟着的元素占据了浮动元素原先的位置。
2. 浮动是个比较特殊的个体，它虽然不在标准文档流中，但是仍然跟标准文档流相互影响。
3. 如果浮动前面的元素没有浮动属性，则浮动会另起一行在此元素的下面浮动。
4. 当一个元素浮动之后，其下方装载文字的容器虽然会占据浮动元素原先的位置，但是其中的文字会一直围绕在浮动元素周围，而不被浮动元素盖住。

#### 2.4 浮动产生的副作用
1. 背景不能显示：如果对父级设置了css背景或者背景图片，而父级不能被撑开，导致背景不能显示
2. 边框不能被撑开
3. margain padding不能正确显示，特别是上下边的margin跟padding不能正确显示

#### 2.5 清除浮动
```
清除浮动：
        clear:left/right/both/none; 
        
        clear 属性规定元素的哪一侧不允许其他浮动元素。
                left：在左侧不允许浮动元素
                right：在右侧不允许浮动元素
                both：在左右两侧均不允许浮动元素
                none：默认值，允许浮动元素出现在两侧。

```