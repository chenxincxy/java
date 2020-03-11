*Java基础**
**1.java内存分析**
	*概念：
		1.栈空间（stack），连续的存储空间，遵循后进先出的原则，用于存放局部变量。
		2.堆空间（heap），不连续的空间，用于存放new出的对象，或者说是类的实例。
		3.方法区（method），方法区在堆空间内，用于存放①类的代码信息；②静态变量和方法；③常量池（字符串敞亮等，具有共享机制）。
		4.Java中除了基本数据类型，其他的均是引用类型，包括类、数组等等。
		5.数据类型的默认值
		基本数据类型默认值：
		数值型：0
		浮点型：0.0
		布尔型：false
		字符型：\u0000
		引用类型：null
		6.变量初始化
		成员变量可不初始化，系统会自动初始化；
		局部变量必须由程序员显式初始化，系统不会自动初始化。
	*


	
**2.自动装箱和拆箱**
	*java的基本数据类型
		1字节：byte和boolean
		2字节:short和char
		4字节：int和float
		8字节：long和double
	*基本数据类型对应的包装类分别是Byte,Boolean,Short,Character,Integer,Float,Long,Double
	*自动装箱
		类的实例化一般
		Object obj=new Object();
		Integer类
		Integer i=5;
		实际上是Integer类自动调用了Integer i=Integer.ValueOf(5);
		基本数据类型装箱完成
	*自动拆箱
		Integer i=new Integer(5);
		int it=i; 这里进行了自动拆箱,调用了i.intValue();把包装类拆成基本数据类型
	*自动装拆箱的注意事项
		`
		Integer i1=10;
		Integer i2=10;
		
		Integer i3=128;
		Integer i4=128;
		
		Integer i5=new Integer(7);
		Integer i6=new Integer(7);
		
		int k1=10;
		int k2=128;
		System.out.println(i1==i2); //true
		System.out.println(i3==i4); //false
		System.out.println(i5==i6); //false
		System.out.println(i1==k1); //true
		System.out.println(i3==k2); //true`
由以上代码可以分析：
	*1.结果第一行打印true，i1和i2被包装后，==比较的是两者的内存地址。相当于分别new了两个对象，应该是false.但是返回是true，为什么呢？
	因为在自动装箱的时候调用了ValueOf()方法，那么我们查看以下该方法的源码：
		` public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }`
	static final int low = -128;
	static final int high = 127;
		-由源码可以ValueOf()源码可以看出,当传入基本数据类型在 -128~127之间返回的是缓存数组里面的数据。第一次声明则将i1放入缓存中，第二次i2直接从缓存数组中取值，没有重新new一个对象。所以两者地址相同，返回true.

		-除了Integer类外，还有Byte、Short、Long、Character也使用了缓存，而Flot、Double没有使用缓存。
	*2.由第一行分析的结果分析，自然可得出第二行结果是false.因为128超出了缓存范围，直接new对象，i3和i4都是新new的对象，放在堆中不同位置。所以==返回false.
	
	*3.由第二行结果分析，自然可得出第三行结果是false.虽然是在low~hight的范围内，但是直接用的new,手动包装，没有调用ValueOf()方法自然没有放缓存和从缓存中取的说法。i5和i6是不同地址

	*4.Integer(无论new否)和int比较，Integer会自动拆箱。两个基本数据类型，==比较的是数值。
	*5.与4同理

	`Integer a = 1;
	Integer b = 2;
	Integer c = 3;
	Integer d = 3;
	 
	Integer e = 321;
	Integer f = 321;
	 
	Long g = 3L;
	Long h = 2L;
	 
	System.out.println(c == d); //true
	
	System.out.println(e == f); //false
	
	System.out.println(c == (a + b));//true
	
	System.out.println(c.equals((a+b))); //true
	System.out.println(g == (a+b));
	System.out.println(g.equals(a+b));
	System.out.println(g.equals(a+h));`

	*1.由之前的分析可知返回true.
	*2.同之前的缓存分析知返回false.
	*3.右边a+b包含了算术运算，因此会触发自动拆箱过程（会调用intValue方法）,==比较符又将左边的自动拆箱，因此它们比较的是数值是否相等。
	反编译 ：c.intValue()==a.intValue()+b.intValue();
	*4.看看Integer中的equals源码
		public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
   		 }
	由此可知，Integer的equals被重写。equals会把同一类型的转化为数值比较。
	因此结果返回true.
	*5.g == (a+b)，首先计算 a+b,也是先调用各自的intValue方法，得到数值之后，==运算符能将隐含的将小范围的数据类型转换为大范围的数据类型，也就是int会被转换成long类型，由于前面的g是Long类型的，也会自动拆箱为long，两个long类型的数值进行比较。因此结果返回true.

	*6.a+b得到的是Integer类型的。Long类中的equals方法和4中的同理，判断括号内的数值是否是Long类型的实例。不是就返回false，不会进行类型转换。因此返回false.

	*7.g.equals(a+h),运算符+会进行类型转换，转为大范围的数据类型。a+h各自拆箱之后是int+long，结果是long，然后long进行自动装箱为Long，两个Long进行equals判断。结果是true.

3.值传递和引用传递
	*java中一直有这样的说法：java中不存在引用传递，只有值传递
	*实际上参数的传递都是传递的是副本。如果是基本数据类型，那么就是基本数据类型的值的拷贝。如果是引用类型，那么传递的就是引用变量值的拷贝。
		*若传入是基本数据类型的值的拷贝。我们可以得出，在被传递该值的方法中对该拷贝值进行改变，不会影响原值。
		*若传入的是引用变量的值的拷贝。因为引用变量的值指向的是引用类型的地址。那么其拷贝值也是指向引用类型的地址。所以我们可以得出，在被传递该值的方法中对该拷贝值进行改变，即是对指向的引用类型进行改变，所以会影响原引用类型的数据。
	#小坑：
		代码1：
		public class test1 {
	public static void main(String[] args) {
		A a=new A();
		a.data=20;
		a.method(a);
		System.out.println("main中的"+a.data);
	}
}
		class A{
				int data=10;
			void method(A a) {
				System.out.println("A中的"+a.data);
			}
		}
 输出：
		A中的20
		main中的20

	  代码2：
	public class test1 {
	public static void main(String[] args) {
		A a=new A();
		a.data=20;
		a.method(a);
		System.out.println("main中的"+a.data);
	}
}
	class A{
			int data=10;
		void method(A a) {
			a=new A();
			System.out.println("A中的"+a.data);
		}
	}
 输出：
	A中的10
	main中的20

	*比较代码1和代码2。
		-代码2中a.data没有改变的原因是：一开始传递的时候，的确传递的是main中改变后的引用变量a的副本。但是在method()方法中a新new了个对象，从一开始指向main中new的20的对象，指向新的对象，所以原本方法中的data还是10;
			

4.为什么hashcode要和equals一起重写？
	*hashcode(散列码)
		Object类中hashCode返回的是对象的地址，所以在Object中一个对象对应一个hashcode,不同对象的hashcode基本上不会相同
	*equals
		Object类中的equals比较的是两个对象的地址。
	*hashcode和equals的关系。
	  -两个对象hashcode比较返回true,equals不一定返回true.
	  -两个对象是同一个对象那么hashcode必须相同。
	*需求：我们想让两个对象的name和age属性相同即为同一个对象。
		*分析：
			1.因为规定两个对象为同一对象，hashcode必须一致。所以首先重写hashcode，使hashcode返回的码是与name和age相关计算得出。
			2.如果重写了的话，由于采用的算法的问题，有可能导致两个不同对象的hashCode相同。在散列表中我们就知道hashcode像一个桶，一个hashcode地址不只有一个对象。我们在hashcode同的情况下，还要在桶里找出两个满足要求的对象。
			3.所以我们要再重写equals方法，使得name和age同即为同一个对象。
		*因此由分析可知Hashset中判断两个对象是否为同一对象的步骤是：
			1.先判断hashcode是否同，同再判断equals是否同，equals同则为同一对象
			2.如果hashcode不同，直接不会再调用equals方法
			因此可知道equals相同的两个对象其hashcode一定一致。又由此可知，重写了equals一定要重写hashcode。如果equals是比较两个对象的name,那么hashcode要散列name.
			但并不代表任何时候equals同则两个对象地址一致。比如在String类中。
		
5.接口和抽象类的区别
	*语法上的区别与共同
		区别
		1.
			抽象类是一个类，可以被子类继承。单继承
			接口不是类，像一种规则。不能被继承，只能被实现。一个类可以实现多个接口。
		2.
			抽象类中不仅可以有抽象方法也可以有具体方法。
			接口只有抽象方法。
		3.
			抽象类的属性可以被各种修饰符修饰。
			接口的属性只能被public static final修饰，或者default
		相同
		1.
			抽象类的子类可以不重写其父类的抽象方法，但是其子类也要变成抽象的。
			同样实现接口的类如果不实现接口中的所有抽象方法，也要变成抽象类。
		2.
			抽象类和接口都不能通过new对象来实例化。
	*设计层面的区别
		抽象类是由下自上的设计思想。由多个子类的共同之处所抽象出来的方法。
		比如有猫，狗，海龟。可以抽象出动物类，抽象方法可以是eat()。不能只有一个猫类，就直接抽象出一个动物类。
		接口是由上而下的设计方法。比如有一个接口是fly.实现它其中的抽象方法的类可以是飞机也可以是鸟。我们定义接口就是一个规则，我们不管谁来实现都要遵守我们定义的规则。

6.访问权限修饰符
	修饰符			同一个类			同一个包		子类		不同包非子类
	public			   √				√		  √			√
	protected		   √				√		  √
	private			   √
	default			   √				√		  √

7.重写和重载的区别
	  -重载是对于一个类中同名方法的多态的体现。
	  同名函数，参数列表不同(参数类型，参数数量，参数顺序),称方法之间发生了重载。
	  重载的方法之间的返回值也可以不同。
	  -重写是在父类和子类之间。子类多态的体现。
	  子类重写父类中的方法，称为重写。保证 方法名一致，返回值一致，参数列表一致。抛出异常一致。
	  
8.static关键字

9.final关键字（final可修饰属性、方法、类）
			-修饰类：当一个类被final修饰时候，表示一个类是终态类，该类不能被继承。
			-修饰方法：当一个方法被final所修饰时，表示该方法是一个终态方法，不能被重写
			-修饰属性：当一个属性被final修饰时，该属性不能被改写。

					注意：（1）	当final修饰一个原生数据类型时候，表示该原生数据类型的值不能发生变化（如：10不能变到20）
						当修饰一个引用类型时，表示该引用类型的地址不能发生变化(即不能再指向其它对象了)，但是该引用
						原本所指的对象的里面内容只要没有final修饰是可以变化的。

					代码示例：
					M m=new M();
					//m.address=new Address();//The final field M.address cannot be assigned
					m.address.name="beijing";
					}
					}
					class M{
					final Address address=new Address();
					
					}
					class Address{
					String name="shanghai";
					}
						（2）对于final类型的成员变量，一般有两种赋初值的方式：（理解记忆）
							a)声明final类型的成员变量的时候就赋上初值
							b)再final类型的成员变量时不赋初值，但再类的【所有】构造方法中都为其赋上初值[如果不赋，构造方法一定会执行其中的一个或几个，那么你必须保证每一个都对final类型变量赋值了，才不会报错]

	  
10.final和finally和finalized的区别
	*final是关键字
	*finally是异常处理中的一种机制，在finally的代码块中的代码一定要执行。
	*finalized是
11.new一个对象的加载顺序(要改进)
	1.先加载父类静态代码块，静态变量，静态方法
	2.加载子类静态代码块，静态变量，静态方法
	3.加载父类非静态代码块
	4.子类非静态代码块
	5.父类构造函数
	6.子类构造函数
	7.父类成员变量和普通方法
	8.子类成员变量和普通方法
12.String、StringBuffered、StringBiulder的区别(可能有错误)
	1.三者都是由final修饰的。StringBuffered、StringBiulder由final修饰为什么还可以对字符串进行改变？
	因为调用append()方法对应的是对象内容上的增加，其对象地址并没有发生改变。
	2.String、StringBuffered是线程安全的。StringBiulder是线程不安全的。
	3.当多线程数据少的时候使用String。多线程大量数据的时候使用StringBuffered。单线程大量数据使用StringBuilder.
	