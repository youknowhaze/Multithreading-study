# 关于多线程相关的练习

## 一、多线程的实现
多线程的实现目前有3种方式，如下
- 继承Thread类，重写run()方法
- 实现Runnable接口，实现run()方法
- 实现Callable<T>接口，实现 T call()方法

在演示过程中加入了文件操作，于是需要引入以下的工具包
```java
<dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.8.0</version>
        </dependency>
```

### 1、继承Thread方法下载网络图片
````java
//具体实现查看此类，主要是演示如何实现及运行多线程
NetPicDownTests.java
````

### 2、实现Runnable接口下载网络图片
````java
//具体实现查看此类，主要是演示如何实现及运行多线程
NetPicDownTests.java
````

### 3、实现Callable<T>接口运行多线程
````java
//具体实现查看此类，主要是演示如何实现及运行多线程
TestCallable.java
````

### 4、龟兔赛跑demo
```java
//共用成员变量需要声明为static静态的
TortoiseHare.java
```

### 5、多线程的静态代理
````java
//静态代理案例，叙述静态代理的实例
StaticProxy.java
````

### 6、线程状态

#### （1）、线程有五大状态
- 新生（new Thread()创建线程的时候）
- 就绪状态 （当调用start方法就进入就绪状态）
- 运行状态  （进入运行状态才会执行线程的代码体）
   - 阻塞状态 （当线程调用sleep或wait方法时进入阻塞状态）
   - dead死亡状态  （线程进入死亡状态意味着线程结束，进入死亡状态的线程不能再次启动）

#### （2）、线程停止
不建议使用stop或destroy等过时或者jdk不建议使用的方法。
**通过参数控制线程的停止**，如下示例所示

```java
public class ThisTest implements Runnable {
    private boolean flag = true;
    @Override
    public void run() {
        while (flag){
            System.out.println("===============");
        }
    }

    public void stopThread(){
        flag = false;
    }


    public static void main(String[] args) throws InterruptedException {
        ThisTest thisTest = new ThisTest();
        Thread t = new Thread(thisTest);
        t.start();
        Thread.sleep(2000);
        thisTest.stopThread();
    }
}
```

#### （3）、线程睡眠
```java
// 睡眠方法
Thread.sleep(1000)
```

####  （4）、线程礼让
当两个线程（线程A和线程B）存在线程礼让时，若线程A设置了礼让，当线程A进入CPU之后，会退出CPU，然后
和线程B重新竞争进入CPU的机会，**注意：线程礼让不意味着下一次一定是B进入CPU，只是意味着多给了B一次竞争的机会**

```java
public class ThisTest implements Runnable {
    @Override
    public void run() {
            System.out.println(Thread.currentThread().getName()+" ===============开始执行");
            Thread.yield();  // 线程礼让，退出线程重新竞争
            System.out.println(Thread.currentThread().getName()+" ===============开始执行");

    }

    public static void main(String[] args) throws InterruptedException {
        ThisTest thisTest = new ThisTest();

        new Thread(thisTest,"A").start();
        new Thread(thisTest,"B").start();

    }
}
```

#### (5)、线程强制执行
```java
Thread t = new Thread();
t.join(); //强制执行官线程
```

#### （6）、观测线程状态
```java
public class ThisTest implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("=================");
    }


    public static void main(String[] args) throws InterruptedException {
        ThisTest thisTest = new ThisTest();
        Thread t = new Thread(thisTest);
        // 观测状态
        System.out.println("*********** "+t.getState()); //new

        // 观察启动后
        t.start(); //启动线程
        System.out.println("*********** "+t.getState());
        //线程执行 --run()中有sleep等待方法
        while (t.getState() != Thread.State.TERMINATED){
            Thread.sleep(100);
            System.out.println("******** "+t.getState());
        }
        //线程死亡
        System.out.println("--------- "+t.getState());

    }
}
```


#### (7)、线程优先级 PRIORITY
PRIORITY用数字表示，范围为1 ~ 10，线程默认优先级为5，如main()方法。
设置线程的优先级超过1 ~ 10的范围会抛出异常
**优先级高的线程不一定先执行，只是意味着机会更大，当优先级低的线程被先执行，这种情况被称为性能倒置**

```java
public class ThisTest implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+" === "+Thread.currentThread().getPriority());
    }


    public static void main(String[] args) throws InterruptedException {
        System.out.println("主线程的优先级=== "+Thread.currentThread().getPriority());

        ThisTest thisTest = new ThisTest();

        Thread t1 = new Thread(thisTest);
        Thread t2 = new Thread(thisTest);
        Thread t3 = new Thread(thisTest);
        Thread t4 = new Thread(thisTest);
        Thread t5 = new Thread(thisTest);

        //先设置优先级再启动，优先级高的线程不一定先执行
        t1.setPriority(2);
        t1.start();
        t2.setPriority(10);
        t2.start();
        t3.start();   //默认优先级为5
        t4.setPriority(12);  // 超出范围抛出异常
        t4.start();
        t5.setPriority(-1);   // 超出范围抛出异常
        t5.start();
    }
}
``` 

#### (8)、守护线程 daemon
当用户进程结束时，他对应的守护进程也会相应结束，可能会有一定的延迟（jvm的关闭需要时间）

````java
public class ThisTest{

    public static void main(String[] args) throws InterruptedException {
        God god = new God();
        Person person = new Person();
        Thread t = new Thread(god);
        t.setDaemon(true); //默认为false，默认用户线程，true为守护线程
        t.start();

        new Thread(person).start();
    }
}


class God implements Runnable{

    @Override
    public void run() {
        while (true){
            System.out.println("===========");
        }
    }
}


class Person implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i < 200; i++) {
            System.out.println("-------------------");
        }
        System.out.println("ending.............");
    }
}
````


