[toc]

# 数据类型

## String

- 常用方法: 单引号, 双引号, 三引号

  | contains()                     | 是否包含特定内容, 返回 true false |
  | ------------------------------ | --------------------------------- |
  | size()    length()             | 字符串长度                        |
  | toString()                     | 转换成string类型                  |
  | indexOf()                      | 元素的索引                        |
  | endsWith()                     | 是否指定字符结尾                  |
  | minus()    plus()              | 去掉, 增加字符串                  |
  | reverse()                      | 反向排序                          |
  | substring(m,n)                 | 截取子串                          |
  | toUpperCase()    toLowerCase() | 大写, 小写转换                    |
  | split()                        | 分隔字符串                        |

## List

- 列表符号: []

- 常用方法

  | +    -    +=    -=       | 元素增加减少         |
  | ------------------------ | -------------------- |
  | add()    <<              | 添加元素             |
  | isEmpty()                | 判断是否为空         |
  | intersect([m,n])         | 取交集               |
  | disjoint([1])            | 取并集               |
  | flatten()                | 合并嵌套的列表       |
  | unique()                 | 去重                 |
  | reverse()                | 翻转                 |
  | sort()                   | 升序                 |
  | count()                  | 元素个数             |
  | join()                   | 将元素按照参数链接   |
  | sum()    min()    max()  | 求和, 最小值, 最大值 |
  | contains()               | 包含特定元素         |
  | remove(n)    removeAll() |                      |
  | each{}                   |                      |

  

# 常用DLS

## Json数据格式化

```groovy
def response = readJSON text: "${scanResult}"
println(scanResult)

// 原生方法
import groovy.json*
    
@NonCPS
def GetJson(text) {
    def prettyJson = JsonOutput.prettyPrint(text)
    new JsonSlurperClassic().parse
}
```

