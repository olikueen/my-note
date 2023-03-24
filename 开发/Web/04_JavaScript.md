### 1. 查看变量类型
```
        console.log(typeof(a));
        console.log(typeof a);
```

### 2. 类型转换
```
        parseInt()          转换成整形
        parseFloat()        转换成浮点型
        var.toString()      转换成字符串
        Boolean()           转换成布尔
```

### 3.
```
        是否是NaN       isNaN(var)
        是否是一个既非Infinity也非NaN的数字
                        isFinite(var)
```

### 4. 弹出输入
```
        var a = prompt("aaaaa");
                接收到的是一个字符串；
```

### 5. 运算
```
        1.  Number + Number  直接数学运算
        2.  任何数与NaN进行运算结果都是NaN;
        3.  Infinity+(-Infinity) = NaN;
        4.  Boolean/Undefined/Null + Number；            自动转换成Number类型，然后进行算术运算；
        5.  String+(Number/Boolean/Undefined/Null)；     +相当于是字符串的拼接符
        
        6.  赋值 =
        7.  自增 自减
        8.  += -= *= /= %= 
        
        9.  > < >= <= == === != !==
        10. 三目运算表达式  var 值 = 表达式 ? 表达式1 : 表达式2
            如果"表达式"的值为真，则整个三目运算表达式的值为"表达式1"的值，否则为"表达式2"的值
        
        11. && 与   || 或   ! 非
```

### 6. if
```
        if (条件) {
            语句;
        }
        
#==============================================
        if (条件) {
            语句;
        } else {
            语句;
        }
        
#==============================================
        if (条件) {
            语句;
        } else if (条件) {
            语句;
        } else {
            语句;
        }
```

### 7. switch
```
        switch (表达式) {
            case 表达式1:
                语句1;
                break;
            case 表达式2:
                语句2;
                break;
            ……
            case 表达式n:
                语句n;
                break;
            default:
                语句d;
        }
```

### 8. while do-while
```
        while (条件) {
            语句;
        }
        
#==============================================
        do {
            语句;
        } while (条件);
```

### 9. for
```
        for (var i = 0; i<10; i++) {
            语句;
        }
        
#==============================================
        for (i in arr) {
            语句;
        }
```

### 10. break continue
```
        break       跳出switch和循环；
        continue    调过本次循环；
```

### 11. 函数
```
        function 函数名(参数列表) {
            语句
            return 表达式;
        }
        
#==============================================
        可以把函数名赋值给变量；
        
#==============================================
        支持匿名函数；
        
#==============================================
        即时函数：这种函数可以在定以后立即调用
            (function(a) {
                console.log("a = ", a)
            }) ("lance is a good man")
```

## 12. 变量提升
```
        //js在执行过程进入新函数时，这个函数内被声明的所有变量都会被移动到函数最开始的地方，叫做变量的提升。
        //注意：仅仅提升变量的声明，不会提升赋值
                var num = 1
                function test_var() {
                    console.log("1--num = " + num)
                    var num = 2
                    //此时打印的值是2，因为函数域始终优先全局域，所以函数内定义的num会覆盖掉所有与它同名的全局变量。
                    console.log("2--num = " + num)  
                }
                test_var()
```

### 13. 数组
```
        1. 使用构造函数创建一个空数组；new 开辟空间并返回内存空间的引用
                var arr1 = new Array();
                
        2. 使用构造函数创建一个容量为20的数组
                var arr2 = new Array(20);
                
        3. 使用构造方法创建一个包含三个元素的数组
                var arr3 = new Array("a", "b", 1);
        4. 通过方括号创建空数组；
                var arr4 = [];
        5. 通过方括号创建包含元素的数组；
                var arr5 = [1, 2, 'a', true]

#==============================================
        遍历
            var arr5 = [1, 3, 'a', true]
            console.log(arr5);
            for (i in arr5) {
                console.log(i +" - " + arr5[i] +" - "+ typeof(arr5[i]))
                console.log(typeof arr5[i])
            }
            
#==============================================
        数组长度                        arr.length
        
        向数组的末尾插入元素            arr.push(item1,item2...)
        向数组的头部插入元素            arr.unshift(item1,item2...)
        删除数组末尾的元素              arr.pop()
        删除数组头部的元素              arr.shift()
        
        用参数字符串将数组中元素拼接成一个新字符串      arr.join(str)
        
        将原数组元素倒置                arr.reverse()
        截取数组元素                    arr.slice(startIndex,endIndex)
        
        在数组中间插入或者删除数组元素，如果要插入元素的话，个数为0         arr.splice(下标,个数,item1,item2...)
        
        将多个数组拼接                  arr1.concat(arr2, arr3...)
        
        从数组的头部查找数组元素，找到返回数组元素的下标，否则话返回-1      arr.indexOf()
        从数组的尾部查找数组元素，找到返回数组元素的下标，否则话返回-1      arr.lastIndexOf()
        
        升序排序                        arr.sort()
        降序排序                        arr.sort(compare)

#==============================================
        多维数组
            arr = [[1, 2, 3], ['a', 'b', 'c']]
```

### 14. 字符串
```
        基本类型字符串      var str1 = "hello"
        引用类型字符串      var str2 = new String("world")
        基本类型的字符串，也可以使用引用类型字符串的属性和方法；

#==============================================
        字符串长度              str.length
        
        获取指定下标的字符                          charAt(index)
        获取指定下标的字符的ASCII码(Unicode)        charCodeAt(index)
        将ASCII码转换成对应的字符                   String.fromCharCode(ASCII码)
        
        字符串大写转换小写      str.toLowerCase()
        字符串小写转换大写      str.toUpperCase()
        
        判断是否相等            ==  值相等；    ===  绝对相等(值和类型都相等)
        
        字符串比较大小          str1.localeCompare(str2)
        
                从左至右依次对比相同下标处的字符，当两个字符不相等时，哪个字符的ASCII值大，那么字符串就大
                返回值为1，左边大于右边，返回值为-1，右边大于左边，返回值为0则相等
        查找字符串      正向查找，返回第一次出现的下标      str.indexOf()
                        反向查找，返回第一次出现的下标      str.lastIndexOf()
                        
        替换子串        str.replace(被替换的子串, 新子串)
        提取子串        str.substring()     str.substr()
        分割字符串      str.split()
        
```

### 15. Date
```
        格林尼治时间(GTM):是英国郊区皇家格林尼治天文台的时间;
            因为地球自转的原因，不同经度上的时间是不相同的，格林尼治天文台是经度为0的地方。
            世界上发生的重大时间都是以格林尼治时间时间为标准的。
            
        世界协调时间(UTC):世界时间。1970年1月1日0点。

#==============================================
        获取当前时间
            var a1 = Date()
            var a2 = new Date()
        
        获取指定时间(如果省略了小时、分钟、秒数，这些会被设置为0)
            var a3 = new Date("2018-12-14")
            var a4 = new Date("2011/11/24")
            var a5 = new Date("2018-9-8")
            var a6 = new Date(2018,10,14,13,15,20)
            var a7 = new Date(10000)
        
#==============================================
        获取日期对象的属性
            获取年份        date.getFullYear()
            获取月份        date.getMonth()     0代表1月
            获取日期        date.getDate()
            获取星期        date.getDay()
            获取小时        date.getHours()
            获取分钟        date.getMinutes()
            获取秒数        date.getSeconds()
            获取毫秒数      date.getMilliseconds()
            
            获取日期对象所表示的日期距离1970-01-01的毫秒数      date.getTime()
            
            设置年份        date.setFullYear(2015)
            设置月份        date.setMonth(8)            传入的月份大于11，则年份增加
            设置日期        date.setDate(29)            如果传入的日期超过了该月应有的天数则会增加月份
            星期一般不用设置
            设置小时        date.setHours(13)           如果传入的值超过23则增加日期
            设置分钟        date.setMinutes(56)         如果传入的值超过了59则增加小时数
            设置秒数        date.setSeconds(10)         传入的值超过59会增加分钟数
            设置毫秒数      date.setMilliseconds(888)   传入的值超过999会增加秒数
            设置距离1970-01-01的毫秒数      date.setTime(1308484904898)
        
#==============================================
        转换成字符串
            包含年月日时分秒        date.toLocaleString()
            包含年月日              date.toLocaleDateString()
            包含时分秒              date.toLocaleTimeString()
            
#==============================================
        返回该日期距离1970年1月1日0点的毫秒数
            Date.parse(dateString)
            
#==============================================
        两个日期对象之间的毫秒差
            var date1 = new Date("2016-10-10 10:10:10");
            var date2 = new Date("2016-10-10 10:10:12");
            console.log(date2 - date1)

```

### 16. 正则
```
        创建正则
            var re1 = /.*?/
            var re2 = new RegExp(".*?")
            
#==============================================
        匹配测试        re.test()
        使用正则分割    str.split(re)
        匹配返回列表    re.exec(str)

```

### 17. JSON
```
        

#==============================================
        

#==============================================


#==============================================


#==============================================


#==============================================


#==============================================
```

###
```

```

###
```

```

###
```

```

###
```

```