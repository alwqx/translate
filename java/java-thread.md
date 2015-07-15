# java多线程基础
##实现线程的两种方法
* 实现Runnable接口

```java
public class Test implements Runnable{
	public void run(){
	//code
	}
}

public static void main(String[] args) {
	Test t = new Test();
	Thread tt = new Thread(t);
	tt.start();
	...
}

```

* 继承Thread,重写`run`方法

```java
public class Test extends Thread{
	public void run9){
	//code
	}
}

public static void main(String[] args) {
	Test t = new Test();
	t.start();
}
```

##Daemon 与 non-Daemon
### Daemon
* 优先级最低的线程,通常来说,当应用程序中没有其它线程运行时,Daemon运行.
* 只有Daemon时,Daemon结束时.JVM会结束程序
* 通常作为普通线程的服务提供者,如GC