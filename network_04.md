# Networking

09.19



물리적인 랜선으로 연결 -> 네트워크 프로그램간의 연결 -> Stream 연결 -> 데이터 주고받기



TCP/IP : 두개의 프로그래밍 간의 커뮤니케이션의 규약



* day03/http package

  ```java
  // Http1.java
  
  package http;
  
  import java.io.BufferedInputStream;
  import java.io.BufferedReader;
  import java.io.InputStream;
  import java.io.InputStreamReader;
  import java.io.StringWriter;
  import java.net.URL;
  import java.net.URLConnection;
  
  public class Http1 {
  
  	public static void main(String[] args) throws Exception{
  		URL url = new URL("http://70.12.60.90/test");
  		URLConnection conn = url.openConnection(); 
  		// 커넥션 연결, 해당 URL로 접속
  		
  		InputStream is = conn.getInputStream();
  		InputStreamReader isr = new InputStreamReader(is);
  //		BufferedInputStream bis = new BufferedInputStream(is);
  		BufferedReader bis = new BufferedReader(isr);
  		
  		String data = null;		
  		StringBuffer sw = new StringBuffer();
  //		int data = 0;
  //		StringWriter sw = new StringWriter();
  //		while((data = bis.read()) != -1){
  		while((data = bis.readLine()) != null) {
  //			char c = (char) data;
  //			sw.write(c);		
  			sw.append(data+"\n");
  		}
  		System.out.println(sw.toString());
  		
  	}
  
  }
  
  //------------------------------------------------------------------------------------

  // Http2.java
  
  package http;
  
  import java.io.BufferedInputStream;
  import java.io.BufferedOutputStream;
  import java.io.FileOutputStream;
  import java.io.InputStream;
  import java.net.URL;
  
  public class Http2 {
  
  	public static void main(String[] args) throws Exception{
  		String urlstr = "http://70.12.60.90/test/oracle.zip";
  		URL url = new URL(urlstr);
  		InputStream is = url.openStream();
  		/*// 밑의 두줄이 위의 한줄
  		URLConnection conn = url.openConnection(); 
  		InputStream is = conn.getInputStream();*/
  		BufferedInputStream bis = new BufferedInputStream(is, 1024);
  		
  		String fileName = "oracle.zip";
  		FileOutputStream fo = new FileOutputStream(fileName);
  		BufferedOutputStream bos = new BufferedOutputStream(fo);
  		
  		int data = 0;
  		while ((data = bis.read()) != -1) {
  //			System.out.println("*"+data);
  			bos.write(data);
  		}
  		System.out.println("Finish..");
  		is.close();
  		bos.flush();
  		bos.close();
  	}
  
  }
  
  //------------------------------------------------------------------------------------
  
  // Http3.java
  
  package http;
  
  import java.io.BufferedReader;
  import java.io.InputStream;
  import java.io.InputStreamReader;
  import java.net.HttpURLConnection;
  import java.net.URL;
  
  public class Http3 {
  
  	public static void main(String[] args) throws Exception{
  		String urlstr = "http://70.12.60.90/test/login.jsp";
  		String id = "idid";
  		String pwd = "11";
  		urlstr += "?id="+id+"&pwd="+pwd;
  		URL url = new URL(urlstr);
  //		InputStream is = url.openStream();
  		
  		HttpURLConnection conn = (HttpURLConnection)url.openConnection();
  		conn.setRequestMethod("POST"); // GET(url 보여서 해킹에 우려)
  		conn.setReadTimeout(5000); // try/catch
  //		conn.getInputStream(); // 서버로 전송
  		InputStream is = conn.getInputStream(); // 서버로 전송
  		// InputStream 생성
  		InputStreamReader isr = new InputStreamReader(is);
  		BufferedReader br = new BufferedReader(isr);
  		
  		if(conn.getResponseCode() == HttpURLConnection.HTTP_OK) {
  			String result = null;
  			while((result = br.readLine()) != null) {
  				System.out.println(result);
  			}
  		}else { // Login Fail
  			System.out.println("Server Down...");
  		}
  		
  		// Http는 갔다오면 끝이라 close 필요 X
  	}
  
  }
  
  
  
  //------------------------------------------------------------------------------------
  
  // Http4.java
  
  package http;
  
  import java.io.BufferedReader;
  import java.io.InputStream;
  import java.io.InputStreamReader;
  import java.net.HttpURLConnection;
  import java.net.URL;
  import java.util.Scanner;
  
  //Main Thread에서 데이터(id,pwd) scanner 입력받고,
  //데이터를 thread로 전송
  class H4 extends Thread{
  	String id;
  	String pwd;
  	public H4(String id, String pwd) {
  		this.id = id;
  		this.pwd = pwd;
  	}
  	
  	public void run() {
  		String urlstr = "http://70.12.60.90/test/login.jsp";
  		urlstr += "?id="+id+"&pwd="+pwd;
  		URL url = null;
  		try {
  			url = new URL(urlstr);
  		} catch (Exception e) {
  			e.printStackTrace();
  		}
  		
  		
  		try {
  			HttpURLConnection conn = (HttpURLConnection)url.openConnection();
  			conn.setRequestMethod("GET");
  			conn.setReadTimeout(5000);
  			InputStream is = conn.getInputStream();
  			InputStreamReader isr = new InputStreamReader(is);
  			BufferedReader br = new BufferedReader(isr);
  			
  			if(conn.getResponseCode() == HttpURLConnection.HTTP_OK) {
  				String result = null;
  				while((result = br.readLine()) != null) {
  					System.out.println(result);
  				}
  			}else { // Login Fail
  				System.out.println("Server Down...");
  			}
  		} catch (Exception e) {
  			e.printStackTrace();
  		}
  		
  	}
  	
  	
  }
  
  
  public class Http4 {
  
  	public static void main(String[] args) {
  		Scanner sc = new Scanner(System.in);
  		while(true) {
  			System.out.println("Input ID/PWD ...");
  			String id = sc.nextLine();
  			String pwd = sc.nextLine();
  //			H4 h4h = new H4();
  //			h4h.setId(id);
  //			h4h.setPwd(id);
  			H4 h4 = new H4(id, pwd);
  			h4.start();
  		}
  		
  	}
  
  }
  
  
  ```
  
  

