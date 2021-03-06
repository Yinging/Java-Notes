### IOC **控制反转**  
工厂模式 java反射
容器控制程序对象之间的关系，而不是传统实现中，由程序代码之间控制，又名 **依赖注入**
ALl类的创建，销毁都是由Spring来控制，也就是说控制对象生命周期的不是引用他的对象，而是Spring。
对于某个对象而言，以前是他控制其他对象，现在是all对象都被spring控制，这就叫控制反转
依赖注入的思想是通过反射机制实现的，在实例化一个类时，它通过反射调用类中的set方法将事先保存在hashmap中的类属性注入到类中。
(依赖注入：在运行期间由容器将依赖关系注入到组件中，就是在运行期，由spring根据配置文件将其他对象的引用通过组将提供的setter方法进行设定)
(控制反转：对组件对象控制权的转移，从程序代码本身转移到了外部容器，通过容器来实现对象组件的装配和管理)

控制反转即IoC (Inversion of Control)，是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。它把传统上由程序代码直接操控的对象的调用权交给容器，通过容器来实现对象组件的装配和管理。所谓的“控制反转”概念就是对组件对象控制权的转移，从程序代码本身转移到了外部容器。

### AOP
代理模式
Spring事先aop:JDK动态代理，cglib动态代理
`JDK`动态代理：其代理对象必须是某个接口的实现，他是通过在运行期间创建一个接口的实现类来完成对目标对象的代理。核心类有InnovationHandler，Proxy
`cglib`动态代理：实现原理类似于JDK动态代理，只是他在运行期间生成的代理对象时针对目标扩展的子类。MethodInterceptor
code Generation Library， 通过字节码底层继承要代理类来实现（如果被代理类被final关键字所修饰，则不行）

1、如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP
2、如果目标对象实现了接口，可以强制使用CGLIB实现AOP
3、如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换

如何强制使用CGLIB实现AOP？
 （1）添加CGLIB库，SPRING_HOME/cglib/* .jar
 （2）在spring配置文件中加入<aop:aspectj-autoproxy proxy-target-class="true"/>

 JDK动态代理和CGLIB字节码生成的区别？
  （1）JDK动态代理只能对实现了接口的类生成代理，而不能针对类
  （2）CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法
    因为是继承，所以该类或方法最好不要声明成final

JDK代理是不需要以来第三方的库，只要要JDK环境就可以进行代理，它有几个要求
* 实现InvocationHandler
* 使用Proxy.newProxyInstance产生代理对象 `newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)`
* 被代理的对象必须要`实现接口`
实现原理
* 通过实现InvocationHandler接口创建自己的调用处理器
* 通过为Proxy类指定ClassLoader对象和一组interface来创建动态代理
* 通过反射机制获取动态代理类的构造函数，其唯一参数类型就是调用处理器接口类型
* 通过构造函数创建动态代理类实例，构造时调用处理器对象作为参数参入
```java
public interface UserManager {    
    public void addUser(String id, String password);    
    public void delUser(String id);    
}

public class UserManagerImpl implements UserManager {    

    public void addUser(String id, String password) {    
        System.out.println("调用了UserManagerImpl.addUser()方法！ ");    

    }    

    public void delUser(String id) {    
        System.out.println("调用了UserManagerImpl.delUser()方法！ ");    

    }    
}  

public class JDKProxy implements InvocationHandler {

	private Object targetObject;// 需要代理的目标对象

	public Object newProxy(Object targetObject) {// 将目标对象传入进行代理
		this.targetObject = targetObject;
		// 返回代理对象
		return Proxy.newProxyInstance(targetObject.getClass().getClassLoader(), targetObject.getClass().getInterfaces(),this);
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args)// invoke方法
			throws Throwable {
		Object ret = null;
		// 调用invoke方法，ret存储该方法的返回值
		System.out.println("执行前...");
		ret = method.invoke(targetObject, args);
		System.out.println("执行后...");
		return ret;
	}
}

main
System.out.println("-----------JDKProxy-------------");
JDKProxy jDKProxy = new JDKProxy();
UserManager userManagerJDK = (UserManager) jDKProxy.newProxy(new UserManagerImpl());
userManagerJDK.addUser("tom", "root");
```



CGLib 必须依赖于CGLib的类库，但是它需要类来实现任何接口代理的是指定的类生成一个子类，覆盖其中的方法，是一种继承但是针对接口编程的环境下推荐使用JDK的代理
在Hibernate中的拦截器其实现考虑到不需要其他接口的条件Hibernate中的相关代理采用的是CGLib来执行。
CGLib实现
```java
public class CGLibProxy implements MethodInterceptor {

	private Object targetObject;// CGLib需要代理的目标对象

	public Object createProxyObject(Object obj) {
		this.targetObject = obj;
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(obj.getClass());
		enhancer.setCallback(this);
		Object proxyObj = enhancer.create();
		return proxyObj;// 返回代理对象
	}

	@Override
	public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
		Object obj = null;
		System.out.println("执行前...");
		obj = method.invoke(targetObject, args);
		System.out.println("执行后...");
		return obj;
	}

}
mainSystem.out.println("-----------CGLibProxy-------------");
CGLibProxy cGLibProxy = new CGLibProxy();
UserManager userManagerCGLib = (UserManager) cGLibProxy.createProxyObject(new UserManagerImpl());
userManagerCGLib.addUser("tom", "root");
```
