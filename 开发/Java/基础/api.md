[toc]
# 1. Scanner
获取键盘输入；

```
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int i = sc.nextInt();   // 从键盘读取一个int
        System.out.println("i: " + i);
        
        int ii = new Scanner(System.in).nextInt();
        System.out.println("ii: " + ii);
    }
```

# 2. Random
生成随机数；

```
    public static void main(String[] args) {
        Random r = new Random();
        int i = r.nextInt();
        System.out.println("生成随机数: " + i);
        
        int ii = r.nextInt(10);     // 生成0-9中的一个随机数
        System.out.println("生成随机数: " + ii);
    }
```

# 3. ArrayList
列表；
```
    public static void main(String[] args) {
        // 创建一个int类型的空列表
        ArrayList<Integer> list = new ArrayList<>();
        System.out.println("list: " + list);    // list: []
    
        // 向list的尾部追加元素
        list.add(1);
        list.add(2);
        list.add(3);
        System.out.println("list: " + list);    // list: [1, 2, 3]
    
        // 通过索引删除元素
        list.remove(1);
        System.out.println("list: " + list);    // list: [1, 3]
    
        // 通过索引查找元素
        int i = list.get(0);
        System.out.println("i: " + i);      // i: 1
    
        // 获取list长度
        System.out.println(list.size());    // 2
    }
```
```
    public static void main(String[] args) {
        // 定义Integer类型数组
        Integer[] i = new Integer[]{1, 2, 3};
        System.out.println("i: " + Arrays.toString(i));     // i: [1, 2, 3]
        // 从数组生成列表
        ArrayList<Integer> l1 = new ArrayList<>(Arrays.asList(i));
        System.out.println("l1: " + l1);    // l1: [1, 2, 3]
        // 从列表生成列表
        ArrayList<Integer> l2 = new ArrayList<>(l1);
        System.out.println("l2: " + l2);    // l2: [1, 2, 3]
        // 列表后追加新列表
        Integer[] i3 = new Integer[] {4,5,6};
        ArrayList<Integer> l3 = new ArrayList<>(Arrays.asList(i3));
        l3.addAll(l1);
        System.out.println("l3 + l1: " + l3);   // l3 + l1: [1, 2, 3, 1, 2, 3]
    }
```

# 4. String
```
    public static void main(String[] args) {
        // String 相当于 char[]
        char[] s1 = new char[]{'a', 'b', 'c'};
        String s2 = new String(s1); // 通过char[] 生成String；
        System.out.println(s1);             // abc
        System.out.println("s1: " + s1);    // s1: [C@6cd8737
        System.out.println("s2: " + s2);    // s2: abc
    }
```
```
// == 比较两个变量对象的地址值
    public static void main(String[] args) {
        String str1 = "abc";
        String str2 = "abc";

        char[] charArry = new char[]{'a','b','c'};
        String str3 = new String(charArry);

        System.out.println(str1 == str2);
        System.out.println(str1 == str3);
        System.out.println(str2 == str3);
    }
```
```
内容比较用 public bollean equals(Object obj)：参数可以是任何对象，只有参数是一个字符串且内容相同才会返回true；
        注意事项：
        1. 任何对象都能用Object接收；
        2. equals方法具有对称性，a.equals(b)和b.equals(a)效果一样；
        3. 如果比较双发一个常量一个变量，推荐吧常量字符串写在前面；
        推荐："abc".equals(str)   防止当变量值是null时，报错空指针异常
        不推荐：str.equals("abc")

public bollean equalsIgnoreCase(Object obj) 忽略大小写，进行内容比较；
```

```
String中与获取相关的常用方法：
    public int length()     获取字符串当中含有的字符串个数，拿到字符串长度；
    public String concat(String str)    将当前字符串个参数字符串拼接成为返回值新的字符串；
    public char charAt(int index)       获取指定索引位置的单个字符；（索引从0开始；）
    public int indexOf(String str)      查找参数字符串在本字符串当中首次出现的索引位置，如果没有找到，返回-1；
    
    public static void main(String[] args) {
        // 获取字符串长度
        int length = "asdadaxczcqweasd".length();
        System.out.println("字符串长度：" + length); // 16
        // 拼接字符串
        String str1 = "Hello";
        String str2 = "World";
        String str3 = str1.concat(str2);
        System.out.println(str1);   // Hello
        System.out.println(str2);   // World
        System.out.println(str3);   // HelloWorld
        // 获取指定索引位置单个字符
        char ch = "Hello".charAt(1);
        System.out.println("在1号索引位置的字符是：" + ch);
        // 查找参数字符串在本来字符串当中第一次出现的位置，如果没有找到返回-1；
        String original = "HelloWorldllo";
        int index = original.indexOf("llo");
        System.out.println("第一次索引位置是：" + index);
        System.out.println("HelloWorld".indexOf("abc"));
    }
```
```
字符串的截取方法：
    public String substring(int index)      截取从参数位置一直字符串末尾，返回新字符串；
    public String substring(int begin, int end)     截取从begin开始，一直到end前结束，中间的字符串；
    备注：[begin, end)，包含左边，不包含右边；
    
    public static void main(String[] args) {
        String str1 = "HelloWorld";
        String str2 = str1.substring(5);
        System.out.println(str1);   // HelloWorld
        System.out.println(str2);   // World

        String str3 = str1.substring(4, 7);
        System.out.println(str3);   // oWo
    }
```
```
String当中与转换相关的常用方法：
    public char[] toCharArray() 将当前字符串拆分成为字符串数组作为返回值；
    public byte[] getBytes()    获得当前字符串底层的字节数组；
    public String replace(CharSequence oldString, CharSequence newString)
        将所有出现的oldString替换为newString，返回替换之后的结果新字符串；
    备注：CharSequence是一个接口类型，意思是说可以接受字符串类型；
    
    public static void main(String[] args) {
        // 转换成为字符数组
        char[] chars = "HelloWorld".toCharArray();
        System.out.println(chars[0]);   // H
        System.out.println(chars.length);   // 10
        // 转换成为字节数组
        byte[] bytes = "abc".getBytes();
        for (int i = 0; i < bytes.length; i++) {
            System.out.println(bytes[i]);
        }

        String str1 = "How do you do ?";
        String str2 = str1.replace("o", "*");
        System.out.println(str1); // How do you do ?
        System.out.println(str2); // H*w d* y*u d* ?
    }
```
```
String当中字符串分割方法：
    public String[] split(String regex) 按照参数的规则将字符串切分成为若干部分；
    注意事项：split方法的参数是一个正则表达式；
    
    public static void main(String[] args) {
        String str1 = "aaa,bbb,ccc";
        String[] array1 = str1.split(",");
        for (int i = 0; i < array1.length; i++) {
            System.out.println(array1[i]);
        }

        String str2 = "aaa bbb ccc";
        String[] array2 = str2.split(" ");
        for (int i = 0; i < array2.length; i++) {
            System.out.println(array2[i]);
        }

        String str3 = "XXX.YYY.ZZZ";
        String[] array3 = str3.split("\\.");    // 必须使用\\. 才能切割成功
        System.out.println(array3.length);  // 0
        for (int i = 0; i < array3.length; i++) {
            System.out.println(array3[i]);
        }
    }
```

# 5. static关键字
```
如果一个成员变量使用了static关键字，那么这个变量不在属于对象自己，而是数据所在的类。多个对象共享一份数据。
```
```
一旦使用static修饰成员方法，那么这就成为了静态方法；
静态方法不属于对象，而是属于类；
如果没有static关键字，那么必须先创建对象，然后通过对象才能使用。
对于静态方法来说，可以通过对象名进行调用，也可以直接通过类名称来调用；推荐使用类名称来调用。

即使使用 对象名.静态方法名称() 来调用，这种写法在编译之后也会被javac翻译为“类名称.静态方法名称()”;

无论是成员变量，还是成员方法，如果有了static，都推荐使用类名称进行调用；
静态变量：类名称.静态变量
静态方法：类名称.静态方法()

本类中的静态方法，可以在本类中，直接使用方法名调用；这种写法在编译之后也会被javac翻译为“本类名称.静态方法名称()”;

注意事项：
1. 静态不能直接访问非静态；
	成员方法，可以访问成员变量；成员方法，可以访问静态变量；
	静态方法，可以访问静态变量；静态方法，不能访问非静态变量；
	原因：因为，在内存中，是现有的静态内容，后有非静态内容；
2. 静态方法中不能使用this；
	原因：this代表当前对象，通过谁调用的方法，谁就是当前对象；
```
```
静态代码块的格式是：
public class 类名称 {
	static {
		// 静态代码块的内容
	}
}

特点：
	当第一次调用本类是，静态代码块执行唯一的一次；
	静态总是优先于非静态，所以静态代码块比构造方法先执行；

```

# 6. Arrays
```
public static String toString(数组)：将参数数组变成字符串（按照默认格式：[元素1, 元素2, 元素3...]）；
public static void sort(数组)：按照升序（从小到大）对数组的元素进行排序；

备注：
1. 如果是数值，sort默认按照升序从小到达；
2. 如果是字符串，sort默认按照字母升序；
3. 如果是自定义类，那么这个自定义类需要有Comparable或者Comparator接口支持；（今后学习）
```

# 7. Math
```
public static double abs(double num): 获取绝对值；
public static double ceil(double num): 向上取整；
public static double floor(double num): 向下取整；
public static long round(double num): 四舍五入；
```