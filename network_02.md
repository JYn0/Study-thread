# Thread

09.18



## 쓰레드의 실행제어

대기중일 때, JVM에 의해 대기상태로 다시 들어옴

* sleep

  Thread.sleep -> 다시 runnable



* interrupt : 진행중인 작업의 쓰레드를 취소할 때

  조건을 걸어서 작업이 끝나게 요청

  쓰레드의 interrupted 상태를 바꾸는 것일 뿐이다.



day02



```java
// Inter1.java

package day02;

import java.util.Scanner;

class Th1 extends Thread{
	public void run() {
		while(!isInterrupted()) { // 내 자신이 interrupted되어있지않으면
			try {
				Thread.sleep(1000); // sleep상태에 interrupt가 걸리면 exception 발생
			} catch (InterruptedException e) {
				e.printStackTrace();
				return;
			}
			System.out.println("Th1");
		}
		System.out.println("Th1 END...");
	}
}

public class Inter1 {

	public static void main(String[] args) {
		Th1 th1 = new Th1();
		th1.start();
		System.out.println("END...");
		System.out.println("END...");
		System.out.println("END...");
		System.out.println("END...");
		
//		try {
//			Thread.sleep(5000);
//		} catch (InterruptedException e) {
//			e.printStackTrace();
//		} // main 5초 멈추있어도 th1은 돌고있음
//		th1.interrupt(); // 5초 지나고 interrupt 걸어서 종료
		
		Scanner sc = new Scanner(System.in);
		System.out.println("Input CMD.....");
		int cmd = sc.nextInt(); // 입력받을때까지 main Thread는 여기에 멈춰있음
		if(cmd == 0) {
			th1.interrupt();
		}		
		sc.close();
	}

}


//---------------------------------------------------------------------------------------


// Suspend.java
package day02;

import java.util.Scanner;

class Sus implements Runnable{

	@Override
	public void run() {
		while(true) {
			System.out.println("-");
//			System.out.println(Thread.currentThread().getName()
//						+ " " +Thread.currentThread().getId());
		}
	}
	
}

public class Suspend {

	public static void main(String[] args) {
		Thread t1 = null;
		Scanner sc = new Scanner(System.in);
		
		while(true) {
			System.out.println("input cmd ?");
			int cmd = sc.nextInt();
			if(cmd == 1) {
				t1 = new Thread(new Sus(),"s1"); // thread 이름은 s1
				t1.start();
			}else if(cmd == 2){
				t1.suspend();
				System.out.println("Suspended");
			}else if(cmd == 3){
				t1.resume();
			}else if(cmd == 4) {
				t1.stop();
			}else if(cmd == 9) {
				return;
			}
		}
	}

}

//---------------------------------------------------------------------------------------

// Suspend2.java

package day02;

import java.util.Scanner;

class Sus2 implements Runnable{

	boolean spd = false; // suspend
	boolean stp = false; // stop
	
	public void setStop() { // 문맥으로 제어하기
		stp = true;
	}
	public void setSus() {
		spd = true;
	}
	public void setRes() {
		spd = false;
	}
	
	@Override
	public void run() {
		while(!stp) {
			if(!spd) {
				System.out.println("-");
			}			
		}
	}
}

public class Suspend2 {

	public static void main(String[] args) {
		
		Thread t1 = null;
		Sus2 sus2 = null;
		Scanner sc = new Scanner(System.in);
		
		while(true) {
			System.out.println("input cmd ?");
			int cmd = sc.nextInt();
			if(cmd == 1) {
				sus2 = new Sus2();
				t1 = new Thread(sus2,"s2");
				t1.start();
			}else if(cmd == 2){
				sus2.setSus();
				System.out.println("Suspended");
			}else if(cmd == 3){
				sus2.setRes();
			}else if(cmd == 4) {
				sus2.setStop();
			}else if(cmd == 9) {
				return;
			}
		}
	}

}


//---------------------------------------------------------------------------------------

// Join.java

package day02;

class Th2 extends Thread{
	int sum;
	
	public int getSum() {
		return sum;
	}
	
	public void run() {
		int i = 1;
		while(!isInterrupted()) {
			sum += i;
			
			if(i == 10) {
				return;
			}
			
			try {
				Thread.sleep(500);
			} catch (InterruptedException e) {
				e.printStackTrace();
				return;
			}
			//System.out.println("Th2: "+i);
			i++;
		}
		System.out.println("Th2 END...");
	}
}

public class Join {

	public static void main(String[] args) {
		Th2 th2 = new Th2();
		System.out.println("Start ...");

		th2.start();
		try {
			th2.join(); // main thread를 멈추게, th2가 끝날때까지 기다려랴
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("sum: "+th2.getSum()); // 0
	}

}


```





Main Thread에서 2개의 Thread를 동작하여 결과의 합을 구한다.

첫번째 Thread는 1~100, 두번째 Thread는 101~150까지의 합

두 Thread의 합을 다시 더해서 결과를 출력 한다.

```java
// Join2.java


package day02;

class count extends Thread{
	int startNum;
	int endNum;
	int sum;
	
	public count(int startNum, int endNum) {
		this.startNum = startNum;
		this.endNum = endNum;
	}
	
	public int result() {
		return sum;
	}
	
	public void run() {
		for(int i=startNum; i<=endNum; i++) {
			sum += i;
		}
	}
}


public class Join2 {

	public static void main(String[] args) {		
		count count1 = new count(1,100);
		count1.start();
		try {
			count1.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		count count2 = new count(101,150);
		count2.start();
		try {
			count2.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		int result = count1.result() + count2.result();
		System.out.println("Thread1 : "+count1.result());
		System.out.println("Thread2 : "+count2.result());
		System.out.println("Result : "+result);
	}

}

```





-----------------------------------------------



## 쓰레드의 동기화(synchronized)

하나의 데이터를 멀티쓰레드가 동시에 접근하여 발생하는 문제를 해결하기 위함

먼저 들어온 thread가 점유한 함수를 다른 thread는 실행 불가

wait에서 기다리다가 notify 알려줌



bank

```java
// Account.java

package bank;

public class Account {
	private int balance;
	
	public Account() {
		
	}
	
	public Account(int balance) {
		this.balance = balance;
	}
	
	public void deposit(int money) throws Exception {
		if(money < 0) {
			throw new Exception();
		}
		balance += money;
		notify(); // wait에서 기다리고있는 애들한테 알려줌
	}
	
	public synchronized void withdraw(int money) throws Exception {
		if(balance >= money) {
			balance -= money;
		}else {
			wait(); // 나머지 thread가 wait 영역으로 들어가서 대기
		}
	}
	
	public int getBalance() {
		return balance;
	}

	// generate to String
	@Override
	public String toString() {
		return "Account [balance=" + balance + "]";
	}

	
	
}


//---------------------------------------------------------------------------------------


// Bank.java

package bank;

class Header extends Thread{
	Account acc;
	public Header(Account acc) {
		this.acc = acc;
	}
	public void run() {
		for(int i=0; i<100; i++) {
			int money = (int)(Math.random()*3+1) * 100;
			try {
				Thread.sleep(10);
				acc.deposit(money);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
}

class Branch extends Thread{
	String name;
	Account acc;
	public Branch(String name, Account acc) {
		this.name = name;
		this.acc = acc;
	}
	
	public void run() {
//		while(acc.getBalance() > 0) { // if
//			int money = (int)(Math.random()*3+1) * 100;
			int money = 500;
			try {
//				Thread.sleep(100);
				acc.withdraw(money);
			} catch (Exception e) {
				e.printStackTrace();
			}
			System.out.println(name+" "+acc.getBalance());
//		}
	}
}


public class Bank {

	public static void main(String[] args) {
		Account acc = new Account(1700);
//		Header h = new Header(acc);
//		h.start();
		
		
		Branch b1 = new Branch("b1", acc);
		Branch b2 = new Branch("b2", acc);
		Branch b3 = new Branch("b3", acc);
		Branch b4 = new Branch("b4", acc);
		b1.start();
		b2.start();
		b3.start();
		b4.start();
	}

}
```



-----------------



main에서 t1에 값을 넣주면 t1 시작(1~100 while) 

T1에서 100을 T2로 던져주면 T2실행 100~1(thread에서 thread 실행)

```java
package ws;

class T1 extends Thread{
	int max;
	String name;
	public T1(int max, String name) {
		this.max = max;
		this.name = name;
	}
	
	public void run() {
		T2 t2 = new T2(max,"T2");
		t2.start();
		
		for(int i=0; i<=max; i++) {
			System.out.println(name + " : " + i);
		}
	}
}

class T2 extends Thread{
	int max;
	String name;
	public T2(int max, String name) {
		this.max = max;
		this.name = name;
	}
	
	public int val(int max) {
		return max;
	}
	
	public void run() {
		for(int i=max; i>=0; i--) {
			System.out.println(name + " : " + i);
		}
	}
}

public class Tt {

	public static void main(String[] args) {
		T1 t1 = new T1(100, "T1");
		t1.start();
	}
	
}

```



 