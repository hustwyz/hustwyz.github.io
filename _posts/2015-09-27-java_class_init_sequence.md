---
layout: post
title: Java类初始化的大体顺序
---

大学就开始用第三方博客写一些技术总结和想法，后来转战Wordpress，但都是写自己的心情想法较多，技术总结也不太多。这两年流行利用Github搭建个人博客，我的Github主页也建立了很久，但是一直没有开动写技术总结。从现在开始坚持写写技术总结吧，今天写第一篇。

前两天在微博上看别人分享携程的一道面试题。题目如下，说出输出的结果：

	public class Base {
    	private String baseName = "base";
	    public Base() {
        	callName();
	    }
    	public void callName() {
    	    System.out.println(baseName);
    	}
 
	    static class Sub extends Base {
        	private String baseName = "sub";
	        public void callName() {
        	    System.out.println (baseName) ;
        	}
    	}
    	
	    public static void main(String[] args) {
        	Base b = new Sub();
    	}
	}
	
看到这个题目有点意思，因为刚不久，我们公司招聘的时候，我就问过来面试的一个类似的问题。虽然具体的代码不一样，但是考得点却是一样的，就是类初始化的大体顺序。

这个题目咋一看，很多人会以为输出结果是sub，我们先来分析创建Sub类实例时代码的执行过程：

>1. 进入Sub类的构造函数<br />
>2. Sub类分配内存<br/>
>3. Base类的构造函数被隐式调用<br />
>4. Base类的构造函数会调用Sub类的callName函数<br />
>5. 执行Sub类的callName函数，打印Sub类的baseName变量的值<br />

按照上面的分析，所以很多人以为结果是sub，但是实际的结果却是null，不信的话可以自己运行一下看看结果。

其实前面的分析基本都是对的，只是忽略了一个很重要的点：成员变量是在什么时候初始化的。

最后输出Sub类的baseName值的时候，baseName的值实际上不是sub，因为执行callName函数的时候baseName成员变量还没有被初始化，所以结果是null。

为什么会这样呢？这个涉及到创建类的实例的时候，类的执行顺序。抛开常量和类的静态变量、静态块的执行顺序不说（先执行父类的静态块、再执行子类的静态块）。实际上这里是先执行父类的构造函数，再进行子类的成员变量初始化的。父类成员变量初始化>父类构造函数>子类成员变量初始化>子类构造函数。加上类的成员变量初始化和构造函数的执行顺序，所以较完整的执行顺序是：

>1. 进入Sub类的构造函数<br />
>2. Sub类分配内存<br/>
>3. 进入Base类的构造函数<br />
>4. Base类的成员变量初始化，baseName的值初始化为base<br />
>5. Base类的构造函数体执行<br />
>6. Base类的构造函数会调用Sub类的callName函数<br />
>7. 执行Sub类的callName函数，打印Sub类的baseName变量的值<br />
>8. Sub类的成员变量初始化，subName的值初始化为sub<br />
>9. Sub类的构造函数体执行<br />

所以最后总结一下：

1. 创建类实例时类的初始化顺序：父类成员变量初始化>父类构造函数>子类成员变量初始化>子类构造函数
2. 尽量避免在类的构造函数里面调用需要被子类重写的函数

最后附上自己在面试的时候，考的题目：

	public class Sample {
	    static class A {
    	    public A(String text) {
        	    System.out.println(text);
        	}
    	}
    
    	static class B extends A {
	        A a2 = new A("2");    
	        static A a1 = new A("1");    
	        public B(String text){
    	        super(text);
        	    a2 = new A("3");
        	}
    	}

	    public static void main(String []args){
    	    B b = new B("4");
    	}
	}
	
这个题目最后的结果是:

1 <br />4<br />2<br />3<br /> 

大家可以自己分析一下执行的过程。



