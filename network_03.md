# I/O

09.19



(받는 곳이 자바가 아닐수 있어서 1bite씩 잘라서 보냄)

Stream으로 데이터를 보낼때는 1bite로 잘라서 보내는게 기본

양쪽이 자바라면 2bite씩 가능



Stream의 기능 확장

필터 : 파이프에 파이프를 씌어서 1bite+1bite해서 2bite 가능

1GB파일을 읽어들일 때, 받는 쪽에서 버퍼를 만들어 큰 사이즈의 데이터를 받아들인다. 



![그림1](https://user-images.githubusercontent.com/50862497/65226033-242eae00-db01-11e9-9a54-e19dcc62977f.png)





* day03 package

  ```java
  // Fi1.java
  
  package day03;
  
  import java.io.BufferedReader;
  import java.io.FileNotFoundException;
  import java.io.FileReader;
  import java.io.IOException;
  
  public class Fi1 {
  
  	public static void main(String[] args) {
          /*
          FileInputStream fi = null;
          try{
          	fi = new FileInputStream("test.txt");
          	int data = 0;
  			while((data=fi.read()) != -1) { // -1: 문장의 끝
  				char c = (char)data; // 1byte씩 읽으면 한글이 깨짐
  				System.out.print(c);
          }   
          */
          
  		FileReader fi = null; // 2byte 전송, 한글 안깨짐
  		BufferedReader br = null; // filter 파이프를 한번더 감싼다
  		try {
  			fi = new FileReader("test.txt");
  			br = new BufferedReader(fi);
  			String data = null;			
  
  			while((data = br.readLine()) != null) {
  				System.out.println(data);
  			}
  			
  		} catch (FileNotFoundException e) {// file 읽을 때
  			e.printStackTrace();
  		} catch (IOException e) {// data=fi.read()
  			e.printStackTrace();
  		} finally {
  			if(fi != null) {
  				try {
  					fi.close();
  				} catch (IOException e) {
  					try {
  						Thread.sleep(2000);
  					} catch (InterruptedException e1) {
  						e1.printStackTrace();
  					}
  					try {
  						fi.close();
  					} catch (IOException e1) {
  						e1.printStackTrace();
  					}
  				}
  			}
  		}
  	}
  }
  
  
  //------------------------------------------------------------------------------------
  
  // Fi2.java
  
  package day03;
  
  import java.io.BufferedInputStream;
  import java.io.BufferedOutputStream;
  import java.io.FileInputStream;
  import java.io.FileOutputStream;
  
  public class Fi2 {
  
  	public static void main(String[] args) throws Exception {
  		FileInputStream fis = null;
  		fis = new FileInputStream("C:\\network\\day03\\test.txt");
  //		System.out.println("available: "+fis.available());
  		BufferedInputStream bis = new BufferedInputStream(fis);
  		
  		FileOutputStream fos = null;
  		fos = new FileOutputStream("test2.txt");
  		BufferedOutputStream bos = new BufferedOutputStream(fos, 5);
  		
  		int data = 0;
  		while((data = bis.read()) != -1) {// 여러 바이트를 한번에 stream으로 읽어들임
  			char c = (char)data;
  			System.out.print(c);
  			bos.write(data);
  		}
  		
  		if(fis != null) {
  			bis.close();
  //			fis.close();
  		}
  		if(fos != null) {
  			bos.flush();
  			bos.close();
  //			fos.close();
  		}
  	}
  
  }
  
  
  //------------------------------------------------------------------------------------
  
  // Fi3.java
  
  package day03;
  
  import java.io.BufferedReader;
  import java.io.BufferedWriter;
  import java.io.FileReader;
  import java.io.FileWriter;
  
  public class Fi3 {
  
  	public static void main(String[] args) throws Exception {
  		FileReader fr = null;
  		fr = new FileReader("C:\\network\\day03\\test.txt");
  		BufferedReader br = new BufferedReader(fr);
  		
  		FileWriter fw = null;
  		fw = new FileWriter("test3.txt");
  		BufferedWriter bw = new BufferedWriter(fw);
  		
  		String data = null;
  		while((data = br.readLine()) != null) {
  			bw.write(data);
  			bw.newLine();
  		}
  		
  		if(fr != null) {
  			br.close();
  		}
  		if(fw != null) {
  			bw.flush();
  			bw.close();
  		}
  		
  	}
  
  }
  
  ```



* Pipe package

  ```java
  // OutputThread.java
  package pipe;
  
  import java.io.IOException;
  import java.io.PipedReader;
  import java.io.PipedWriter;
  import java.util.Scanner;
  
  public class OutputThread extends Thread {
  
  	PipedWriter output = new PipedWriter();
  
  	public OutputThread(String name) {
  		super(name); // thread의 이름
  		output = new PipedWriter();
  	}
  
  	// scanner 입력 받아서 전송
  	public void run() {
  		Scanner sc = new Scanner(System.in);
  		try {
  //			String msg = "OutputThread... Hello";
  			String msg = sc.nextLine();
  			System.out.println("Sent : " + msg);
  			output.write(msg);
  			output.close(); // finally에서 null체크해주기
  		} catch (IOException e) {
  			e.printStackTrace();
  		}
  		sc.close();
  	}
  
  	public PipedWriter getOutput() {
  		return output;
  	}
  
  	public void connect(PipedReader input) { // output에 input을 연결
  		try {
  			output.connect(input);
  		} catch (IOException e) {
  			e.printStackTrace();
  		}
  	}
  
  }
  
  //------------------------------------------------------------------------------------
  
  //InputThread.java
  
  package pipe;
  
  import java.io.BufferedReader;
  import java.io.IOException;
  import java.io.PipedReader;
  import java.io.PipedWriter;
  
  public class InputThread extends Thread{
  	PipedReader input;
  	BufferedReader br;
  
  	
  	public InputThread(String name) {
  		super(name);
  		input = new PipedReader();
  		br = new BufferedReader(input);
  	}
  	
  	public void run() {
  //		int data = 0;
  		String data = null;
  //		StringWriter sw = new StringWriter();
  		StringBuffer sb = new StringBuffer(); // append 가능
  		System.out.println("Ready");
  		try {
  //			while((data = input.read()) != -1) { // input.read에서 입력 기다림
  			while((data = br.readLine()) != null) {
  //				sw.write(data);
  				sb.append(data);
  			}
  //			System.out.println("Received : "+sw.toString());
  			System.out.println("Received : "+sb.toString());
  		} catch (IOException e) {
  			e.printStackTrace();
  		}
  	}
  	
  	public PipedReader getInput() {
  		return input;
  	}
  	
  	public void connect(PipedWriter output) {
  		try {
  			input.connect(output);
  		} catch (IOException e) {
  			e.printStackTrace();
  		}
  	}
  }
  
  
  //------------------------------------------------------------------------------------
  
  // Main.java
  
  package pipe;
  
  public class Main {
  
  	public static void main(String[] args) {
  		InputThread it = new InputThread("inThread");
  		OutputThread ot = new OutputThread("outThread");
  		
  		it.connect(ot.getOutput()); // inputThread에 outThread를 넣어주어 접속을 시킨다
  		
  		it.start();
  		ot.start();
  		
  	}
  
  }
  
  ```



* ser package

  ```java
  // User.java
  
  package ser;
  
  import java.io.Serializable;
  
  public class User implements Serializable {
  	private String id;
  	transient private String pwd; // -> pwd는 null로 전송
  	private int age;
  		
  	public User() {
  		super();
  	}
  
  	public User(String id, String pwd, int age) {
  		super();
  		this.id = id;
  		this.pwd = pwd;
  		this.age = age;
  	}
  
  	public String getId() {
  		return id;
  	}
  
  	public void setId(String id) {
  		this.id = id;
  	}
  
  	public String getPwd() {
  		return pwd;
  	}
  
  	public void setPwd(String pwd) {
  		this.pwd = pwd;
  	}
  
  	public int getAge() {
  		return age;
  	}
  
  	public void setAge(int age) {
  		this.age = age;
  	}
  
  	@Override
  	public String toString() {
  		return "User [id=" + id + ", pwd=" + pwd + ", age=" + age + "]";
  	}
  	
  }
  
  //------------------------------------------------------------------------------------
  
  // Main.java
  
  package ser;
  
  import java.io.BufferedOutputStream;
  import java.io.FileOutputStream;
  import java.io.ObjectOutputStream;
  
  public class Main {
  // object를 stream으로 보내기
  	public static void main(String[] args) throws Exception{
  		FileOutputStream fos = new FileOutputStream("user.dat");
  		BufferedOutputStream bos = new BufferedOutputStream(fos);
  		ObjectOutputStream oos = new ObjectOutputStream(bos);
  		// object를 보내기위해 buffer 한번더
  		
  		User user = new User("id01", "pwd01", 20);
  		oos.writeObject(user); // 보내기
  		
  		oos.close(); // 마지막것만 close하면 된다
  		
  	}
  
  }
  
  
  //------------------------------------------------------------------------------------
  
  // Main2.java
  
  package ser;
  
  import java.io.BufferedInputStream;
  import java.io.FileInputStream;
  import java.io.ObjectInputStream;
  
  public class Main2 {
  
  	public static void main(String[] args) throws Exception {
  		FileInputStream fis = new FileInputStream("C:\\network\\day03\\user.dat");
  		BufferedInputStream bis = new BufferedInputStream(fis);
  		ObjectInputStream ois = new ObjectInputStream(bis);
  		
  		User user = new User();
  		user = (User)ois.readObject();
  		System.out.println(user.toString());
  		
  	}
  
  }
  
  ```

  

* http package

  ```java
  // Http1.java
  package http;
  
  import java.io.BufferedInputStream;
  import java.io.InputStream;
  import java.io.StringWriter;
  import java.net.URL;
  import java.net.URLConnection;
  
  public class Http1 {
  
  	public static void main(String[] args) throws Exception{
  		URL url = new URL("http://70.12.60.90/test");
  		URLConnection conn = url.openConnection(); 
  		// 커넥션 연결, 해당 URL로 접속
  		
  		InputStream is = conn.getInputStream();
  		BufferedInputStream bis = new BufferedInputStream(is);
  		
  		int data = 0;
  		StringWriter sw = new StringWriter();
  		while((data = bis.read()) != -1){
  			char c = (char) data;
  			sw.write(c);			
  		}
  		System.out.println(sw.toString());
  		
  	}
  
  }
  
  
  ```



