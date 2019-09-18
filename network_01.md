# Thread

09.17



java project

day01



자바는 멀티쓰레드를 지원한다



```java
public static void main(String[] args) {
	제어는할수있지만, 다른 프로세스는 불가
}
```

프로세스 : 실행 중인 프로그램

쓰레드 : 프로세스 안에서 여러 쓰레드 돌리는 거

멀티 쓰레드 : 메모리를 작게 사용(장점), CPU사용률은 여러개의 프로세스가 떠있는것과 동일

멀티 프로세싱(태스킹) : 크롬, 브라우저, 이클립스 등등 돌아가는것 / 코어 하나에 잠깐씩 돌아가면서

멀티 쓰레드 : 하나의 자원안에서 여러개의 쓰레드가 같이 돌아가는것, 실행이 여러개가 돈다

CPU가 듀얼이면 독립적으로 두개가 사용

쓰레드 인보케이션 방식





```java
// Th1.java

package day01;
// P732 그림13-6 (b) 

// main안에서 thread 돌리기
class MyThread extends Thread{ // 1. thread에서 상속받을수있다
	// thread는 멤버variable과 ~를 가질수있다
	String name;
	
	public MyThread(String name) {
		this.name = name;
	}
	
	@Override
	public void run() { // 3. thread는 run에서 실행
		for(int i=0; i<30; i++) {
			try {
				Thread.sleep(3);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			yield(); // t3(priority 최대)가 동시다발적으로 도는 것을 방지
			System.out.println(name+" : "+i);
		}
		
	}	
}

public class Th1 {

	public static void main(String[] args) { // process안에서 main thread
		MyThread t1 = new MyThread("T1"); // name : T1
		MyThread t2 = new MyThread("T2");
		MyThread t3 = new MyThread("T3");
		
		t1.setPriority(5);
		t2.setPriority(3);
		t3.setPriority(10);
		// max는 10
		
		t1.start(); // run 실행
		t2.start();
		t3.start();
		
	}

}


//-----------------------------------------------

// Th2.java

package day01;
// java는 싱글 inheritance
class MyThread2 implements Runnable{ // 인터페이스를 통해서 thread를 만들 수 있음
	String name;
	
	public MyThread2(String name) {
		this.name = name;
	}
	
	@Override
	public void run() { // 3. thread는 run에서 실행
		for(int i=0; i<30; i++) {
			try {
				Thread.sleep(400);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(name+" : "+i);
		}
		
	}	
}

public class Th2 {

	public static void main(String[] args) { // process안에서 main thread
		Thread t1 = new Thread(new MyThread2("T1"));
		Thread t2 = new Thread(new MyThread2("T2"));
		t1.start();
		t2.start();
	}

}

```



```java
// Th3.java

package day01;

public class Th3 {

	public static void main(String[] args) {
		Runnable r = new Runnable() {
			
			@Override
			public void run() {
				for(int i=0; i<100; i++) {
					try {
						Thread.sleep(3);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println("R : " +i);
				}
			}
		};
		
		Runnable r2 = new Runnable() {
			
			@Override
			public void run() {
				for(int i=0; i<100; i++) {
					try {
						Thread.sleep(3);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println("R2 : " +i);
				}
			}
		};
		ThreadGroup tg1 = new ThreadGroup("TG1");
		tg1.setMaxPriority(3);
		// 두개의 thread는 하나로 묶임 -> r과 r2를 tg1으로 묶음
		new Thread(tg1,r,"th1").start();
		new Thread(tg1,r2,"th1").start();
//		new Thread(r).start();
		
		
	}

}

```



```java
// Th4.java

package day01;

class SaveThread extends Thread{
	public void run() {
		while(true) {
			try {
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			save();
		}
	}
	
	public void save() {
		System.out.println("SAVE..");
	}
}

public class Th4 {
	
	public static void main(String[] args) {
		SaveThread st = new SaveThread();
		st.setDaemon(true); // thread의 무한루프  main 끝나면 자동으로 종료
		st.start();
		for(int i=0; i<20; i++) { 
			try {
				Thread.sleep(500);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(i);
		}
	}
}

```





-----------------------

Th5.java

Scanner 입력 받은 숫자까지 for loop를 이용하여 출력한다.

```java
package day01;

import java.util.Scanner;

class scThread extends Thread{
	String name;
	int num;
	public scThread(String name, int num) {
		this.name = name;
		this.num = num;
	}
	
	public void run() {		
		for(int i=0; i<num; i++) {
			try {
				Thread.sleep(500);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(name+" : "+i);
		}	
	}
}

public class Th5 {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		
		int num = sc.nextInt();
		scThread sct = new scThread("SCT1", num);
		sct.start();
		
		num = sc.nextInt();
		scThread sct2 = new scThread("SCT2", num);	
		sct2.start();
		
		sc.close();
	}

}

```

