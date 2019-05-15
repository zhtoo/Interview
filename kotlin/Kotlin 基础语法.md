## Kotlin 基础语法

#### 包声明 

    package com.xxx.xxxx

#### 类导入

    import java.xxx.*

#### 函数

1、无参无返回

    fun initData(){
   		 //...
    }

2、有参无返回

 	fun initData(a: Int){//a 参数名称  Int 参数类型 
   		 //...
    }

3、无参有返回

 	 fun isEquals(): String {//返回类型 String
        return "Hello World"
    }


4、有参有返回

 	 fun isEquals(a: String, b: String): Boolean {// 返回类型 Boolean
        return a.equals(b)
    }

PS:
1、无返回值的函数(类似Java中的void)的返回参数为 Unit，可以省略，一般不写。
2、表达式作为函数体，返回类型自动推断：

    fun sum(a: Int, b: Int) = a + b

 	//public 方法则必须明确写出返回类型
    public fun sum(a: Int, b: Int): Int = a + b

### 










