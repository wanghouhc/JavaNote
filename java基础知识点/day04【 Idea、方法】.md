# day04【 Idea、方法】

## 今日内容

- 集成开发工具IDEA
- 方法的参数和返回值
- 方法的重载

## 教学目标

- [ ] 掌握idea开发工具的安装
  - [ ] 下一步....
- [ ] 能够使用idea创建Java工程
  - [ ] 略
- [ ] 能够使用idea创建包和类
  - [ ] 包创建在src目录下,类创建在包下
  - [ ] 包:
  - [ ] 一般都是小写
  - [ ] 规范:公司域名倒过来写+项目名+功能名
  - [ ] 多级包使用点隔开
- [ ] 了解idea的Java工程目录
  - [ ] src:用来存放源文件的 .java文件
  - [ ] out:用来存放编译之后的 .class文件
- [ ] 了解idea字体的设置
  - [ ] setting-font--设置
- [ ] 掌握idea常用开发快捷键
  - [ ] 略
- [ ] 了解项目的导入和删除
  - [ ] 导入: 先拷贝模块到项目路径下,然后再导入
  - [ ] 删除: remove\delete进行删除,注意需要进入文件夹路径手动删除文件
- [ ] 掌握方法定义的参数和返回值含义
  - [ ] 参数:用来接收调用者传入的实际数据
  - [ ] 返回值:把方法计算的结果返回给调用者
- [ ] 掌握方法的调用过程
  - [ ] 调用方法
  - [ ] 加载方法
  - [ ] 声明形参
  - [ ] 把实参赋值给形参
  - [ ] 执行方法体
  - [ ] return接收方法,把返回值返回给调用者
- [ ] 掌握方法的三种调用形式
  - [ ] 直接调用
  - [ ] 赋值调用
  - [ ] 输出调用
- [ ] 掌握方法定义的注意事项
  - [ ] 方法定义在类中,方法外
  - [ ] 方法不调用不会执行
  - [ ] 方法的执行顺序与定义位置无关,与调用顺序有关
  - [ ] return语句后面不能再写代码,因为写了也是无效代码
  - [ ] 返回值类型与返回值的类型一致
- [ ] 掌握方法重载的概念
  - [ ] 在同一个类类中,出现多个同名的方法,但是参数列表不同,与返回值类型和修饰符无关
- [ ] 能够判断出方法的重载
  - [ ] 是否在同一个类中
  - [ ] 是否是同名的方法
  - [ ] 是否参数列表不同: 参数个数不同,参数类型不同,参数顺序不同

# 第一章 开发工具IntelliJ IDEA 

## 1.1 开发工具概述

### 目标:

​	之前咋们开发程序都是使用记事本开发,缺点:效率低,没有提示等,所以今天介绍一款高级开发工具idea

### 步骤:

### 讲解:

​	IDEA是一个专门针对Java的集成开发工具(IDE)，由Java语言编写。所以，需要有JRE运行环境并配置好环境变量。它可以极大地提升我们的开发效率。可以自动编译，自动检查错误。在公司中，使用的就是IDEA进行开发。

### 小结:

eclipse 集成开发工具  免费软件

Intellij Idea 集成开发工具  收费软件  功能比较强大,目前市场上都是使用它

## 1.2 IDEA软件安装

### 目标:

​	掌握idea开发工具的安装

### 步骤:

- 安装idea开发工具

### 讲解:

此软件集成了32位和64位，双击`ideaIU-2017.3.2.exe` 进入安装。

1. 欢迎界面

![](img/idea1.jpg)

2. 选择安装路径

![](img/idea2.jpg)

3. 配置安装选项

![](img/idea3.jpg)

4. 开始菜单

![](img/idea4.jpg)

5. 安装完毕

![](img/idea5.jpg)

IDEA开发工具安装完成

### 小结:

略

## 1.3 IDEA首次驱动和创建第一个java项目

### 目标:

​	idea首次驱动和创建第一个java项目

### 步骤:

- idea首次驱动
- 创建第一个java项目

### 讲解:

1. 选择不导入任何设置，点击`OK` 

![](img/idea6.jpg)

2. 选择 `Create New Project` 

![](img/idea7.jpg)

3. 点击`new` 按钮，配置安装的`JDK9` 版本

![](img/idea8.jpg)

选择`  JDK9` 目录，选择 `Empty Project` ，点击确定

![](img/idea9.jpg)

![](img/idea10.jpg)

4. 不使用模板

![](img/idea11.jpg)

5. 为工程起名字 `demo` ，并存储到`d:\ideawork\demo` 目录下，如果d盘没有这个目录，会自动创建。

![](img/idea12.jpg)

6. 打开一个每日一帖对话框，勾掉每次启动显示，点击`close` 

![](img/idea13.jpg)

7. IDEA的工作界面，我们的项目已经创建好了，如果再新建项目，点击`File->new->Project` 

![](img/idea14.jpg)

### 小结:

略

## 1.4 创建包和类

### 目标:

​	创建包和类

### 步骤:

- 创建包
- 创建类

### 讲解:

0. 包取名:
   1. 包名都是小写字母
   2. 规范:包名一般都是公司域名反过来写+[项目名]+功能名
   3. 包其实就是一个文件夹
   4. 多级包使用点分割

1. 展开创建的工程，在源代码目录`src` 上，鼠标右键，选择`new->package` ，键入包名`com.itheima.demo` ，点击确定。

![](img/idea15.jpg)

![](img/idea16.jpg)

​	右键点击`com.itheima.demo` ，选择 `Show in Explorer` ，会发现创建包的目录结构。

![](img/idea16_2.jpg)

​	可见`com.itheima.demo` ，表示创建了多级的文件夹。

> 小贴士：所谓包，就是文件夹，用来对类文件进行管理。

2. 在创建好的包上，鼠标右键，选择 `new->class`  创建类，键入类名。

![](img/idea17.jpg)

![](img/idea18.jpg)

3. 在代码编辑区，键入主方法，并输出`HelloWorld` 。

![](img/idea19.jpg)

4. 运行程序，在代码编辑区鼠标右键，选择`Run HelloWorld` 即可，或在菜单中选择`Run->Run HelloWorld ` 。

![](img/idea20.jpg)

### 小结:

略

## 1.5 字体设置

### 目标:

​	设置idea的字体

### 步骤:

- 设置idea的字体

### 讲解:

IDEA工具的默认字体非常小，代码编辑器和控制台的输出字体都需要进行调整。

* 点击菜单栏上的`File->Settings->Editor->Font`修改字体。

![](img/idea21.jpg)

![](img/idea22.jpg)

### 小结:

略

## 1.6 IDEA的项目目录

### 目标:

​	idea项目目录的介绍

### 步骤:

- idea项目目录的介绍

### 讲解:

- 我们创建的项目，在d:\ideawork目录的demo下
  - `out`目录是存储编译后的.class文件
  - `src` 目录是存储我们编写的.java源文件

  ![](img/idea24.jpg)

### 小结:

略

## 1.7 IDEA常用快捷键

| 快捷键               | 功能                                   |
| -------------------- | -------------------------------------- |
| `Alt+Enter`          | 导入包，自动修正代码                   |
| `Ctrl+Y`             | 删除光标所在行                         |
| `Ctrl+D`             | 复制光标所在行的内容，插入光标位置下面 |
| `Ctrl+Alt+L`         | 格式化代码                             |
| `Ctrl+/`             | 单行注释                               |
| `Ctrl+Shift+/`       | 选中代码注释，多行注释，再按取消注释   |
| `Alt+Shift+上下箭头` | 移动当前代码行                         |

## 1.8 IDEA修改快捷键

在IDEA工具中，`Ctrl+空格`的快捷键，可以帮助我们补全代码，但是这个快捷键和Windows中的输入法切换快捷键冲突，需要修改IDEA中的快捷键。

`File->Settings->keymap->Main menu->code->Completion->Basic`

![](img/idea25.jpg)

双击`Basic->remove->Ctrl+空格`

![](img/idea26.jpg)

再次双击`Basic->Add Keyboard->键入 Alt+/->点击OK`

![](img/idea28.jpg)

## 1.9 IDEA导入和关闭项目

### 目标:

​	掌握idea导入和关闭项目

### 步骤:

- 关闭项目
- 导入项目

### 讲解:

关闭IDEA中已经存在的项目，`File->Close Project`

![](img/idea29.jpg)

`File->Close Project `这时IDEA回到了刚启动界面，点击项目上的`X`，IDEA中就没有这个项目了

![](img/idea31.jpg)

在IDEA的启动界面上，点击`OPEN` ，选择项目目录即可

![](img/idea32.jpg)

> 小贴士：
>
> 课后若想通过IDEA同时开启多个项目，点击OPEN打开项目时，点击New Window按钮

![](img/idea33.jpg)

### 小结:

略

# 第二章 方法(定义方法的格式,定义方法,调用方法)

## 2.1 回顾--方法的定义和调用

### 目标:

​	回顾无参数无返回值方法的定义和调用

### 步骤:

- 无参数无返回值方法的定义
- 无参数无返回值方法的调用

### 讲解:

前面的课程中，使用过嵌套循环输出矩形，控制台打印出矩形就可以了，因此将方法定义为`void`，没有返回值。在主方法`main` 中直接被调用。

```java
public class Method_Demo1 {
    public static void main(String[] args) {
        print();
    }

    private static void print() {
        for (int i = 0; i < 5; i++) {
            for (int j = 0; j < 8; j++) {
                System.out.print("*");
            }
            System.out.println();
        }
    }
}

```

`print`方法被`main`方法调用后直接输出结果，而`main`方法并不需要`print`方法的执行结果，所以被定义为`void`。

### 小结:

略

## 2.2 定义方法的格式详解

### 目标:

​	掌握定义方法的格式

### 步骤:

- 定义方法的格式
- 格式的解释说明

### 讲解:

```java
修饰符 返回值类型 方法名(数据类型 变量名,数据类型 变量名,...){
        //代码省略...
        return 结果\返回值;
}
```

- 修饰符： public static 固定写法
- 返回值类型： 表示方法运行的结果的数据类型，方法执行后将结果返回到调用者
- 方法名:自己定义的名称,符合标识符命名规则和规范即可
- 参数列表：方法在运算过程中的未知数据，调用者调用方法时传递
- return：将方法执行后的结果带给调用者，方法执行到`return` ，整体方法运行结束

> 小贴士：return **结果**;  这里的"结果"在开发中，我们正确的叫法成为**方法的返回值**

### 小结:

略

## 2.3 定义方法的两个明确

### 目标:

​	掌握定义方法的步骤

### 步骤:

- 定义方法的步骤

### 讲解:

- **需求：**定义方法实现两个整数的求和计算。
  - **明确返回值类型**：方法计算的是整数的求和，结果也必然是个整数，返回值类型定义为int类型。
  - **明确参数列表**：计算哪两个整数的和，并不清楚，但可以确定是整数，参数列表可以定义两个int类型的变量，由调用者调用方法时传递

```java
public class Method_Demo2 {
    public static void main(String[] args) {
        
        // 调用方法getSum，传递两个整数，这里传递的实际数据又称为实际参数
        // 并接收方法计算后的结果，返回值     
        int sum = getSum(5, 6);
        System.out.println(sum);
    }

  	/*
    定义计算两个整数和的方法
    返回值类型，计算结果是int
    参数：不确定数据求和，定义int参数.参数又称为形式参数
    */
    public static int getSum(int a, int b) {
        return a + b;
    }
}

```

程序执行，主方法`main` 调用`getSum` 方法，传递了实际数据`5和6` ，两个变量`a和b`接收到的就是实际参数，并将计算后的结果返回，主方法`main`中的变量`sum`接收的就是方法的返回值。

### 小结:

略

## 2.4 调用方法的流程图解  

### 目标:

​	掌握方法的执行流程

### 步骤:

- 方法执行的流程图

### 讲解:

赋值调用      数据类型 变量名 = 方法名(参数);

![](img/方法调用图解.jpg) 

### 小结:

略

## 2.5 定义方法练习

### 目标:

​	巩固方法的定义和调用

### 步骤:

- 练习一
- 练习二
- 练习三

#### 练习一

**比较两个整数是否相同**

- 分析：定义方法实现功能，需要有两个明确，即`返回值` 和 `参数列表`。
  - **明确返回值**：比较整数，比较的结果只有两种可能，相同或不同，因此结果是布尔类型，比较的结果相同为true。
  - **明确参数列表**：比较的两个整数不确定，所以默认定义两个int类型的参数。

```java
public class Method_Demo3 {
    public static void main(String[] args) {
        //调用方法compare，传递两个整数
        //并接收方法计算后的结果，布尔值
        boolean bool = compare(3, 8);
        System.out.println(bool);
    }

    /*
        定义比较两个整数是否相同的方法
        返回值类型，比较的结果布尔类型
        参数：不确定参与比较的两个整数
    */
    public static boolean compare(int a, int b) {
        if (a == b) {
            return true;
        } else {
            return false;
        }
    }
}

```

#### 练习二

**计算1+2+3...+100的和**

- 分析：定义方法实现功能，需要有两个明确，即`返回值`和`参数`。 
  - **明确返回值**：1~100的求和，计算后必然还是整数，返回值类型是int
  - **明确参数**：需求中已知到计算的数据，没有未知的数据，不定义参数

```java
public class Method_Demo4 {
    public static void main(String[] args) {
        //调用方法getSum
        //并接收方法计算后的结果，整数
        int sum = getSum();
        System.out.println(sum);
    }

    /*
        定义计算1~100的求和方法
        返回值类型，计算结果整数int
        参数：没有不确定数据
    */
    public static int getSum() {
        //定义变量保存求和
        int sum = 0;
        //从1开始循环，到100结束
        for (int i = 1; i <= 100; i++) {
            sum = sum + i;
        }
        return sum;
    }
}

```

#### 练习三

**实现不定次数打印**

- 分析：定义方法实现功能，需要有两个明确，即`返回值`和`参数`。 
  - **明确返回值**：方法中打印出`HelloWorld`即可，没有计算结果，返回值类型`void`。 
  - **明确参数**：打印几次不清楚，参数定义一个整型参数

```java
public class Method_Demo5 {
    public static void main(String[] args) {
        //调用方法printHelloWorld，传递整数
        printHelloWorld(9);
    }

    /*
          定义打印HelloWorld方法
          返回值类型，计算没有结果 void
          参数：不确定打印几次
    */
    public static void printHelloWorld(int n) {
        for (int i = 0; i < n; i++) {
            System.out.println("HelloWorld");
        }
    }
}

```

## 2.6 定义方法的注意事项

### 目标:

​	定义方法的注意事项

### 步骤:

- 定义方法的注意事项

### 讲解:

- 定义位置，类中方法外面。
- 返回值类型，必须要和`return`语句返回的类型相同，否则编译失败 。

```java
// 返回值类型要求是int
public static int getSum() {
    return 5;// 正确，int类型
    return 1.2;// 错误，类型不匹配
    return true;// 错误，类型不匹配
}
```

- 不能在`return` 后面写代码，`return` 意味着方法结束，所有后面的代码永远不会执行，属于无效代码。

```java
public static int getSum(int a,int b) {
  	return a + b;
  	System.out.println("Hello");// 错误，return已经结束，这里不会执行，无效代码
}
```

```
2.方法不调用不会执行
3.方法的执行顺序与定义位置无关,与调用顺序有关
```

### 小结:

- 定义位置，类中方法外面。
- 返回值类型，必须要和`return`语句返回的类型相同，否则编译失败 。
- 不能在`return` 后面写代码，`return` 意味着方法结束，所有后面的代码永远不会执行，属于无效代码。

## 2.7 调用方法的三种形式

### 目标:

​	调用方法的三种形式

### 步骤:

- 直接调用
- 赋值调用
- 输出调用

### 讲解:

- **直接调用：**直接写方法名调用

```java
public static void main(String[] args) {
  	print();
}
public static void print() {
 	System.out.println("方法被调用");
}
```

- **赋值调用：**调用方法，在方法前面定义变量，接收方法返回值

```java
public static void main(String[] args) {
  	int sum = getSum(5,6);
  	System.out.println(sum);
}
public static int getSum(int a,int b) {
  	return a + b;
}
```

- **输出语句调用：**

  - 在输出语句中调用方法，`System.out.println(方法名())` 。

  ```java
  public static void main(String[] args) {
    	System.out.println(getSum(5,6));
  }
  public static int getSum(int a,int b) {
    	return a + b;
  }
  ```

  - 不能用输出语句调用`void` 类型的方法。因为方法执行后没有结果，也就打印不出任何内容。

  ```java
  public static void main(String[] args) {
    	System.out.println(printHello());// 错误，不能输出语句调用void类型方法
  }
  public static void printHello() {
    	System.out.println("Hello");
  }
  ```

### 小结:

略

## 2.8 方法重载

### 目标:

​	掌握方法的重载

### 步骤:

- 方法重载的概念
- 演示方法重载
- 方法重载的注意事项

### 讲解:

- **方法重载**：指在同一个类中，允许存在一个以上的同名方法，只要它们的参数列表不同即可，与修饰符和返回值类型无关。
- 参数列表不同：个数不同，数据类型不同，顺序不同。
- 重载方法调用：JVM通过方法的参数列表，调用不同的方法。

### 小结:

略

## 2.9 方法重载练习

### 目标:

​	熟练掌握方法重载

### 步骤:

- 练习一
- 练习二
- 练习三

### 讲解:

#### 练习一

比较两个数据是否相等。参数类型分别为两个`byte`类型，两个`short`类型，两个`int`类型，两个`long`类型，并在`main`方法中进行测试。 

```java
public class Method_Demo6 {
    public static void main(String[] args) {
        //定义不同数据类型的变量
        byte a = 10;
        byte b = 20;
        short c = 10;
        short d = 20;
        int e = 10;
        int f = 10;
        long g = 10;
        long h = 20;
        // 调用
        System.out.println(compare(a, b));
        System.out.println(compare(c, d));
        System.out.println(compare(e, f));
        System.out.println(compare(g, h));
    }

    // 两个byte类型的
    public static boolean compare(byte a, byte b) {
        System.out.println("byte");
        return a == b;
    }

    // 两个short类型的
    public static boolean compare(short a, short b) {
        System.out.println("short");
        return a == b;
    }

    // 两个int类型的
    public static boolean compare(int a, int b) {
        System.out.println("int");
        return a == b;
    }

    // 两个long类型的
    public static boolean compare(long a, long b) {
        System.out.println("long");
        return a == b;
    }
}

```

#### 练习二

判断哪些方法是重载关系。

```java
public static void open(){}
public static void open(int a){}
static void open(int a,int b){}// 冲突
public static void open(double a,int b){}
public static void open(int a,double b){}// 冲突
public void open(int i,double d){}// 冲突
public static void OPEN(){}// 不是重载
public static void open(int i,int j){}// 冲突
```

#### 练习三

模拟输出语句中的`println`方法效果，传递什么类型的数据就输出什么类型的数据，只允许定义一个方法名`println`。 

```java
public class Method_Demo7 {
	public static void println(byte a) {
        System.out.println(a);
    }

    public static void println(short a) {
        System.out.println(a);
    }

    public static void println(int a) {
        System.out.println(a);
    }

    public static void println(long a) {
        System.out.println(a);
    }

    public static void println(float a) {
        System.out.println(a);
    }

    public static void println(double a) {
        System.out.println(a);
    }

    public static void println(char a) {
        System.out.println(a);
    }

    public static void println(boolean a) {
        System.out.println(a);
    }

    public static void println(String a) {
        System.out.println(a);
    }
}
```

### 小结:

略