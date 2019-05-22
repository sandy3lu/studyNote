# 简介
ThreadLocal用于保存某个线程共享变量：对于同一个static ThreadLocal，不同线程只能从中get，set，remove自己的变量，而不会影响其他线程的变量。
1、ThreadLocal.get: 获取ThreadLocal中当前线程共享变量的值。
2、ThreadLocal.set: 设置ThreadLocal中当前线程共享变量的值。
3、ThreadLocal.remove: 移除ThreadLocal中当前线程共享变量的值。
4、ThreadLocal.initialValue: ThreadLocal没有被当前线程赋值时或当前线程刚调用remove方法后调用get方法，返回此方法值。

# 原理
线程共享变量缓存如下：
Thread.ThreadLocalMap<ThreadLocal, Object>;
1、Thread: 当前线程，可以通过Thread.currentThread()获取。
2、ThreadLocal：我们的static ThreadLocal变量。
3、Object: 当前线程共享变量。
我们调用ThreadLocal.get方法时，实际上是从当前线程中获取ThreadLocalMap<ThreadLocal, Object>，然后根据当前ThreadLocal获取当前线程共享变量Object。

这种存储结构的好处：
1、线程死去的时候，线程共享变量ThreadLocalMap则销毁。
2、ThreadLocalMap<ThreadLocal,Object>键值对数量为ThreadLocal的数量，一般来说ThreadLocal数量很少，相比在ThreadLocal中用Map<Thread, Object>键值对存储线程共享变量（Thread数量一般来说比ThreadLocal数量多），性能提高很多。

# 例子
```java
public class MyThreadLocal 
{
    private static final ThreadLocal<Object> threadLocal = new ThreadLocal<Object>(){
        /**
         * ThreadLocal没有被当前线程赋值时或当前线程刚调用remove方法后调用get方法，返回此方法值
         */
        @Override
        protected Object initialValue()
        {
            System.out.println("调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！");
            return null;
        }
    };
     
    public static void main(String[] args)
    {
        new Thread(new MyIntegerTask("IntegerTask1")).start();
        new Thread(new MyStringTask("StringTask1")).start();
        new Thread(new MyIntegerTask("IntegerTask2")).start();
        new Thread(new MyStringTask("StringTask2")).start();
    }

    public static class MyIntegerTask implements Runnable
    {
        private String name;
         
        MyIntegerTask(String name)
        {
            this.name = name;
        }
 
        @Override
        public void run() 
        {
            for(int i = 0; i < 5; i++)
            {
                // ThreadLocal.get方法获取线程变量
                if(null == MyThreadLocal.threadLocal.get())
                {
                    // ThreadLocal.et方法设置线程变量
                    MyThreadLocal.threadLocal.set(0);
                    System.out.println("线程" + name + ": 0");
                }
                else
                {
                    int num = (Integer)MyThreadLocal.threadLocal.get();
                    MyThreadLocal.threadLocal.set(num + 1);
                    System.out.println("线程" + name + ": " + MyThreadLocal.threadLocal.get());
                    if(i == 3)
                    {
                        MyThreadLocal.threadLocal.remove();
                    }
                }
                try
                {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
            }   
        }
         
    }
     
    public static class MyStringTask implements Runnable
    {
        private String name;
         
        MyStringTask(String name)
        {
            this.name = name;
        }
 
        @Override
        public void run() 
        {
            for(int i = 0; i < 5; i++)
            {
                if(null == MyThreadLocal.threadLocal.get())
                {
                    MyThreadLocal.threadLocal.set("a");
                    System.out.println("线程" + name + ": a");
                }
                else
                {
                    String str = (String)MyThreadLocal.threadLocal.get();
                    MyThreadLocal.threadLocal.set(str + "a");
                    System.out.println("线程" + name + ": " + MyThreadLocal.threadLocal.get());
                }
                try
                {
                    Thread.sleep(800);
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
            }
        }
         
    }

}

```
