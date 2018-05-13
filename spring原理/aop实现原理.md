# aop实现原理
## 代理模式
### 实现方式
#### 静态代理
##### 实现方式

* 继承

      public class Client {
      	public static void main(String[] args) {
      		DriverCar car = new DriverCar();
      		car.move();
      	}
      }

      class DriverCar extends Car
      {
      	public void move(){
      		System.out.println("开始开车............");
      		super.move();
      		System.out.println("开车结束");
      	}
      }

      class Car {
      	public void move(){
      		System.out.println("小车正在行驶中......");
      		try {
      			long sleepTime = (long) ((Math.random()*1000)+2000);
      			System.out.println("行车了:"+sleepTime+"毫秒");
      			Thread.sleep(sleepTime);
      		} catch (InterruptedException e) {
      			e.printStackTrace();
      		}
      	}
      }

* 组合

      public class ComponentProxy {
      	public static void main(String[] args) {
      		DriverCar car = new DriverCar1(new Car());
      		car1.move();
      	}
      }

      class DriverCar{

      	Car car ;

      	public DriverCar(Car car) {
      		this.car = car;
      	}

      	public void move(){
      		System.out.println("开始开车............");
      		car.move();
      		System.out.println("开车结束");
      	}

      }



#### 动态代理
##### 实现方式
* JDK动态代理
  1.	创建一个实现接口InvacationHandler的类，它必须实现invoke（）方法
  2.	创建被代理的类以及接口
  3.	调用Proxy的静态方法，创建一个代理类



        public interface Movable {
      	 public void move();
        }
        public class Car implements Movable{
        	public void move(){
        		System.out.println("小车正在行驶中......");
        		try {
        			long sleepTime = (long) ((Math.random()*1000)+2000);
        			System.out.println("行车了:"+sleepTime+"毫秒");
        			Thread.sleep(sleepTime);
        		} catch (InterruptedException e) {
        			e.printStackTrace();
        		}
        	}
        }
        public class Handler implements InvocationHandler {
        	private Object target;
        	// 构造方法，给我们要代理的真实对象赋初值
        	public Handler(Object target) {
        		this.target = target;
        	}
        	/**
        	 * proxy: 指代我们所代理的那个真实对象
        	 * method: 指代的是我们所要调用真实对象的某个方法的Method对象
        	 * args:指代的是调用真实对象某个方法时接受的参数
        	 */
        	@Override
        	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        		// TODO Auto-generated method stub
        		System.out.println("开始行车");
        		method.invoke(target, args);
        		System.out.println("行车结束");
        		return null;
        	}
        }
        public class Client {
        	public static void main(String[] args) {
        		Car car = new Car();
        		// 我们要代理哪个真实对象，就将该对象传进去，最后是通过该真实对象来调用其方法的
        		InvocationHandler handler = new Handler(car);
        		/*
        		 * 通过Proxy的newProxyInstance方法来创建我们的代理对象，我们来看看其三个参数
        		 * 第一个参数handler.getClass().getClassLoader()，
        		 * 我们这里使用handler这个类的ClassLoader对象来加载我们的代理对象
        		 * 第二个参数realSubject.getClass().getInterfaces()
        		 * 我们这里为代理对象提供的接口是真实对象所实行的接口
        		 * 表示我要代理的是该真实对象，这样我就能调用这组接口中的方法了
        		 * 第三个参数handler， 我们这里将这个代理对象关联到了上方的
        		 * InvocationHandler 这个对象上
        		 */
        		Movable newCar = (Movable) Proxy.newProxyInstance(handler.getClass().getClassLoader(), car.getClass().getInterfaces(), handler);
        		newCar.move();
        	}
        }


* 动态代理源码解析（基于jdk1.8）
    1. 生成动态接口类的方法：Proxy.newProxyInstance()


        public static Object newProxyInstance(ClassLoader loader,  
                                          Class<?>[] interfaces,  
                                          InvocationHandler h)  
        throws IllegalArgumentException {  
          if (h == null) {  
              throw new NullPointerException();  
          }  
          final Class<?>[] intfs = interfaces.clone();  
          final SecurityManager sm = System.getSecurityManager();  
          if (sm != null) {  
              checkProxyAccess(Reflection.getCallerClass(), loader, intfs);  
          }  
          // 这里是生成class的地方  
          Class<?> cl = getProxyClass0(loader, intfs);  
          // 使用我们实现的InvocationHandler作为参数调用构造方法来获得代理类的实例  
          try {  
              final Constructor<?> cons = cl.getConstructor(constructorParams);  
              final InvocationHandler ih = h;  
              if (sm != null && ProxyAccessHelper.needsNewInstanceCheck(cl)) {  
                  return AccessController.doPrivileged(new PrivilegedAction<Object>() {  
                      public Object run() {  
                          return newInstance(cons, ih);  
                      }  
                  });  
              } else {  
                  return newInstance(cons, ih);  
              }  
          } catch (NoSuchMethodException e) {  
              throw new InternalError(e.toString());  
          }  
      }  



其中newInstance只是调用Constructor.newInstance来构造相应的代理类实例，这里重点是看getProxyClass0这个方法的实现：

    private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces
        //代理接口的数量不能超过65535
        if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }

         // JDK对代理进行了缓存，如果已经存在相应的代理类，则直接返回，否则才会通过ProxyClassFactory来创建代理  
        return proxyClassCache.get(loader, interfaces);
    }

  其中代理缓存是使用WeakCache实现的，如下

    private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
          proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());

  具体的缓存逻辑这里暂不关心，只需要关心ProxyClassFactory是如何生成代理类的，ProxyClassFactory是Proxy的一个静态内部类，实现了WeakCache的内部接口BiFunction的apply方法：

    private static final class ProxyClassFactory
        implements BiFunction<ClassLoader, Class<?>[], Class<?>>
    {
        // 代理类名称前缀
        private static final String proxyClassNamePrefix = "$Proxy";

        // 用于生成代理类名字的计数器  
        private static final AtomicLong nextUniqueNumber = new AtomicLong();

        @Override
        public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

             // 省略验证代理接口的代码……  

            String proxyPkg = null;     //定义产生代理类包
            int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

            // 对于非公共接口，代理类的包名与接口的相同
            for (Class<?> intf : interfaces) {
                int flags = intf.getModifiers();
                if (!Modifier.isPublic(flags)) {
                    accessFlags = Modifier.FINAL;
                    String name = intf.getName();
                    int n = name.lastIndexOf('.');
                    String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                    if (proxyPkg == null) {
                        proxyPkg = pkg;
                    } else if (!pkg.equals(proxyPkg)) {
                        throw new IllegalArgumentException(
                            "non-public interfaces from different packages");
                    }
                }
            }
             // 对于公共接口的包名，默认为com.sun.proxy  
            if (proxyPkg == null) {
                // if no non-public proxy interfaces, use com.sun.proxy package
                proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
            }

           // 获取计数  
            long num = nextUniqueNumber.getAndIncrement();
            // 默认情况下，代理类的完全限定名为：com.sun.proxy.$Proxy0，com.sun.proxy.$Proxy1……依次递增  
            String proxyName = proxyPkg + proxyClassNamePrefix + num;
             // 这里才是真正的生成代理类的字节码的地方  
            byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                proxyName, interfaces, accessFlags);
            try {
              // 根据二进制字节码返回相应的Class实例
                return defineClass0(loader, proxyName,
                                    proxyClassFile, 0, proxyClassFile.length);
            } catch (ClassFormatError e) {
                throw new IllegalArgumentException(e.toString());
            }
        }
    }

  ProxyGenerator是sun.misc包中的类，它没有开源，但是可以反编译来一探究竟：

    public static byte[] generateProxyClass(final String var0, Class[] var1) {  
      ProxyGenerator var2 = new ProxyGenerator(var0, var1);  
      final byte[] var3 = var2.generateClassFile();  
      // 这里根据参数配置，决定是否把生成的字节码（.class文件）保存到本地磁盘，我们可以通过把相应的class文件保存到本地，再反编译来看看具体的实现，这样更直观  
      if(saveGeneratedFiles) {  
          AccessController.doPrivileged(new PrivilegedAction() {  
              public Void run() {  
                  try {  
                      FileOutputStream var1 = new FileOutputStream(ProxyGenerator.dotToSlash(var0) + ".class");  
                      var1.write(var3);  
                      var1.close();  
                      return null;  
                  } catch (IOException var2) {  
                      throw new InternalError("I/O exception saving generated file: " + var2);  
                  }  
              }  
          });  
      }  
      return var3;  
    }  

  saveGeneratedFiles这个属性的值从哪里来呢：

    private static final boolean saveGeneratedFiles = ((Boolean)AccessController.doPrivileged(new GetBooleanAction("sun.misc.ProxyGenerator.saveGeneratedFiles"))).booleanValue();  

  GetBooleanAction实际上是调用Boolean.getBoolean(propName)来获得的，而Boolean.getBoolean(propName)调用了System.getProperty(name)，所以我们可以设置sun.misc.ProxyGenerator.saveGeneratedFiles这个系统属性为true来把生成的class保存到本地文件来查看。
这里要注意，当把这个属性设置为true时，生成的class文件及其所在的路径都需要提前创建，否则会抛出FileNotFoundException异常。


  即我们要在运行当前main方法的路径下创建com/sun/proxy目录，并创建一个$Proxy0.class文件，才能够正常运行并保存class文件内容。
反编译$Proxy0.class文件，如下所示：

  可以看到，动态生成的代理类有如下特性：
    1. 继承了Proxy类，实现了代理的接口，由于java不能多继承，这里已经继承了Proxy类了，不能再继承其他的类，所以JDK的动态代理不支持对实现类的代理，只支持接口的代理。
    2. 提供了一个使用InvocationHandler作为参数的构造方法。
    生成静态代码块来初始化接口中方法的Method对象，以及Object类的equals、hashCode、toString方法。
    3. 重写了Object类的equals、hashCode、toString，它们都只是简单的调用了InvocationHandler的invoke方法，即可以对其进行特殊的操作，也就是说JDK的动态代理还可以代理上述三个方法。
    4. 代理类实现代理接口的sayHello方法中，只是简单的调用了InvocationHandler的invoke方法，我们可以在invoke方法中进行一些特殊操作，甚至不调用实现的方法，直接返回。

编译之后的代码

    import com.imooc.pattern.Subject;
    import java.lang.reflect.InvocationHandler;
    import java.lang.reflect.Method;
    import java.lang.reflect.Proxy;
    import java.lang.reflect.UndeclaredThrowableException;

    public final class $Proxy0
      extends Proxy
      implements Subject
    {
      private static Method m1;
      private static Method m3;
      private static Method m2;
      private static Method m4;
      private static Method m0;

      public $Proxy0(InvocationHandler paramInvocationHandler)
        throws
      {
        super(paramInvocationHandler);
      }

      public final boolean equals(Object paramObject)
        throws
      {
        try
        {
          return ((Boolean)this.h.invoke(this, m1, new Object[] { paramObject })).booleanValue();
        }
        catch (Error|RuntimeException localError)
        {
          throw localError;
        }
        catch (Throwable localThrowable)
        {
          throw new UndeclaredThrowableException(localThrowable);
        }
      }

      public final void hello()
        throws
      {
        try
        {
          this.h.invoke(this, m3, null);
          return;
        }
        catch (Error|RuntimeException localError)
        {
          throw localError;
        }
        catch (Throwable localThrowable)
        {
          throw new UndeclaredThrowableException(localThrowable);
        }
      }

      public final String toString()
        throws
      {
        try
        {
          return (String)this.h.invoke(this, m2, null);
        }
        catch (Error|RuntimeException localError)
        {
          throw localError;
        }
        catch (Throwable localThrowable)
        {
          throw new UndeclaredThrowableException(localThrowable);
        }
      }

      public final void request()
        throws
      {
        try
        {
          this.h.invoke(this, m4, null);
          return;
        }
        catch (Error|RuntimeException localError)
        {
          throw localError;
        }
        catch (Throwable localThrowable)
        {
          throw new UndeclaredThrowableException(localThrowable);
        }
      }

      public final int hashCode()
        throws
      {
        try
        {
          return ((Integer)this.h.invoke(this, m0, null)).intValue();
        }
        catch (Error|RuntimeException localError)
        {
          throw localError;
        }
        catch (Throwable localThrowable)
        {
          throw new UndeclaredThrowableException(localThrowable);
        }
      }

      static
      {
        try
        {
          m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { Class.forName("java.lang.Object") });
          m3 = Class.forName("com.imooc.pattern.Subject").getMethod("hello", new Class[0]);
          m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
          m4 = Class.forName("com.imooc.pattern.Subject").getMethod("request", new Class[0]);
          m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
          return;
        }
        catch (NoSuchMethodException localNoSuchMethodException)
        {
          throw new NoSuchMethodError(localNoSuchMethodException.getMessage());
        }
        catch (ClassNotFoundException localClassNotFoundException)
        {
          throw new NoClassDefFoundError(localClassNotFoundException.getMessage());
        }
      }
    }
aop是基于jdk动态代理实现,具体动态带的实现也可以参考jdk去理解
