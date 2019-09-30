# TCP/IP

09.23

day 05

* tcp1 package

  1:1채팅

  ```java
  // Server.java
  package tcp1;
  
  import java.io.DataInputStream;
  import java.io.DataOutputStream;
  import java.io.IOException;
  import java.io.InputStream;
  import java.io.OutputStream;
  import java.net.ServerSocket;
  import java.net.Socket;
  
  class ServerThread extends Thread{
  	Socket socket;
  	OutputStream out;
  	DataOutputStream dout;
  	
  	InputStream in;
  	DataInputStream din;
  	
  	public ServerThread(Socket socket) throws IOException { // Socket을 만들어서 Stream
  		this.socket = socket;
  		out = socket.getOutputStream();
  		dout = new DataOutputStream(out);
  		
  		in = socket.getInputStream();
  		din = new DataInputStream(in);
  	}
  	
  	public void run() {
  		try {
  			String str = null;
  			str = din.readUTF();
  			System.out.println(socket.getInetAddress()+":"+str);
  			
  			dout.writeUTF("Hi");
  		} catch (IOException e) {
  			e.printStackTrace();
  		} finally {
  			if(din != null) {
  				try {
  					din.close();
  				} catch (IOException e) {
  					e.printStackTrace();
  				}
  			}
  			if(dout != null) {
  				try {
  					dout.close();
  				} catch (IOException e) {
  					e.printStackTrace();
  				}
  			}
  			if(socket != null) {
  				try {
  					socket.close();
  				} catch (IOException e) {
  					e.printStackTrace();
  				}
  			}
  		}
  		
  	}
  }
  
  public class Server { // Server는 소켓만 만든다
  	
  	boolean flag = true; // accept를 위한 while 루프 안에서 사용
  	
  	ServerSocket serverSocket;
  	
  	public Server() {
  		
  	}
  	
  	public Server(int port) throws IOException {
  		serverSocket = new ServerSocket(port); // ServerSocket을 port(몇번)로 하겠다
  		System.out.println("Server Start..");
  	}
  
  	public void start() throws IOException{
  		while(flag) {
  			System.out.println("Server Ready..");
  			Socket socket = serverSocket.accept(); // 사용자들이 들어올때마다 socket만들기
  			System.out.println(socket.getInetAddress());
  			new ServerThread(socket).start();
  		}
  		System.out.println("Server End..");
  	}
  	
  	public static void main(String[] args) {
  		Server server = null;
  		try {
  			server = new Server(8888);
  			server.start();
  		} catch (IOException e) {
  			e.printStackTrace();
  		}
  	}
  
  }
  
  //------------------------------------------------------------------------------------
  
  // Client.java
  package tcp1;
  
  import java.io.DataInputStream;
  import java.io.DataOutputStream;
  import java.io.InputStream;
  import java.io.OutputStream;
  import java.net.Socket;
  
  public class Client {
  	
  	Socket socket;
  	InputStream in;
  	DataInputStream din;
  	
  	OutputStream out;
  	DataOutputStream dout;
  	
  	public Client() {
  		
  	}
  	
  	public Client(String ip, int port) {
  		boolean flag = true;
  		while(flag) {
  			try {
  				socket = new Socket(ip, port);
  				if(socket != null && socket.isConnected()) {
  					break;
  				}
  			} catch (Exception e) { // 서버가 안켜져있으면
  				System.out.println("Re-Try");
  				try {
  					Thread.sleep(3000);
  				} catch (InterruptedException e1) {
  					e1.printStackTrace();
  				}
  			} 
  		}
  	}
  	
  	public void start() throws Exception{
  		try {
  			out = socket.getOutputStream();
  			dout = new DataOutputStream(out);
  			
  			in = socket.getInputStream();
  			din = new DataInputStream(in);
  			
  			dout.writeUTF("~.~");
  			
  			String str = din.readUTF();
  			System.out.println(str);
  		} catch (Exception e){
  			throw e;
  		} finally {
  			if(dout != null) {
  				dout.close();
  			}
  			if(din != null) {
  				din.close();
  			}
  			if(socket != null) {
  				socket.close();
  			}
  		}
  	}
  	
  	public static void main(String[] args) {
  		Client client = null;
  		client = new Client("70.12.60.90",8888);
  		try {
  			client.start();
  		} catch (Exception e) {
  			e.printStackTrace();
  		}
  	}
  
  }
  
  
  ```

  
  
  
  
  ```java
  // Server.java
  package tcp1;
  
  import java.io.DataInputStream;
  import java.io.DataOutputStream;
  import java.io.IOException;
  import java.io.InputStream;
  import java.io.OutputStream;
  import java.net.ServerSocket;
  import java.net.Socket;
  import java.util.Collection;
  import java.util.HashMap;
  import java.util.Iterator;
  import java.util.Map;
  import java.util.Scanner;
  
  public class Server { // Server는 소켓만 만든다
  
  	boolean flag = true; // accept를 위한 while 루프 안에서 사용
  	boolean rflag = true;
  	Map<String, DataOutputStream> map = new HashMap<>(); // ip, port
  	Map<String, String> map2 = new HashMap<>(); // ip, id
  	ServerSocket serverSocket;
  
  	public Server() { }
  
  	public Server(int port) throws IOException {
  		serverSocket = new ServerSocket(port); // ServerSocket을 port(몇번)로 하겠다
  		System.out.println("Server Start..");
  		Runnable r = new Runnable() {
  			@Override
  			public void run() {
  				try {
  					 while (flag) {
  						System.out.println("Server Ready..");
  						Socket socket = serverSocket.accept();
  						new Receiver(socket).start();
  						System.out.println(socket.getInetAddress());
  					} 
  				} catch (IOException e) {
  					e.printStackTrace();
  				}
  			}
  		};
  		new Thread(r).start();
  	}
  
  	public void start() throws IOException {
  		Scanner sc = new Scanner(System.in);
  		boolean sflag = true;
  		while (sflag) {
  			System.out.println("Input Msg.");
  			String str = sc.next();
  			sendMsg(str);
  			if (str.equals("q")) {
  				break;
  			}
  		}
  		System.out.println("Bye....");
  		sc.close();
  	}
  
  	public void sendMsg(String msg) {
  		Sender sender = new Sender();
  		sender.setMsg(msg);
  		sender.start();
  	}
  
  	// Inner Class
  	class Sender extends Thread {
  
  		String msg;
  
  		public void setMsg(String msg) {
  			this.msg = msg;
  		}
  
  		public void run() {
  			Collection<DataOutputStream> col = map.values(); // values : key값 무시하고 value 꺼낼수있음
  			Iterator<DataOutputStream> it = col.iterator();
  			while (it.hasNext()) {
  				try {
  					it.next().writeUTF(msg);
  				} catch (IOException e) {
  					e.printStackTrace();
  				}
  			}
  		}
  	}
  
  	class Receiver extends Thread {
  		Socket socket;
  		InputStream in;
  		DataInputStream din;
  		OutputStream out;
  		DataOutputStream dout;
  		
  		String ip;
  
  		public Receiver() {
  
  		}
  
  		public Receiver(Socket socket) throws IOException {
  			this.socket = socket;
  			in = socket.getInputStream();
  			din = new DataInputStream(in);
  
  			out = socket.getOutputStream();
  			dout = new DataOutputStream(out);
  			ip = socket.getInetAddress().toString();
  			map.put(ip, dout);
  			System.out.println("접속자수:"+map.size());
  		}
  
  		public void run() {
  			try {
  				while (rflag) {
  					String str = din.readUTF();
  //					System.out.print(socket.getInetAddress()+": ");
  					if(socket.getInetAddress().toString().equals("/70.12.60.108")) {
  						System.out.print("지연 : ");
  					}else if(socket.getInetAddress().toString().equals("/70.12.60.106")) {
  						System.out.print("재영 : ");
  					}else if(socket.getInetAddress().toString().equals("/70.12.60.99")) {
  						System.out.print("지훈 : ");
  					}else {
  						System.out.print(socket.getInetAddress()+": ");
  					}
  					if(str.equals("q")) {
  						map.remove(ip);
  						System.out.println("Out");
  						System.out.println("접속자수:"+map.size());
  						break;
  					}
  					System.out.println(str);
  					sendMsg(str);
  				}
  				if(socket != null) {
  					socket.close();
  				}
  			} catch (Exception e) {
  				e.printStackTrace();
  			}
  		}
  	}
  
  	public static void main(String[] args) {
  		Server server = null;
  		try {
  			server = new Server(8888);
  			server.start();
  		} catch (IOException e) {
  			e.printStackTrace();
  		}
  	}
  
  }
  
  
  // Client.java
  package chat;
  
  import java.io.DataInputStream;
  import java.io.DataOutputStream;
  import java.io.IOException;
  import java.io.InputStream;
  import java.io.OutputStream;
  import java.net.Socket;
  import java.util.Scanner;
  
  
  // socket만들고 reciver 받을준비, scanner로 입력받아 key in 하면 sender에 의해 메시지 보내기
  public class Client {
  
  	Socket socket;
  	boolean rflag = true;
  
  	public Client() {
  
  	}
  
  	public Client(String ip, int port) throws IOException {
  		boolean flag = true;
  		while (flag) {
  			try {
  				socket = new Socket(ip, port);
  				if (socket != null && socket.isConnected()) {
  					break;
  				}
  			} catch (Exception e) { // 서버가 안켜져있으면
  				System.out.println("Re-Try");
  				try {
  					Thread.sleep(3000);
  				} catch (InterruptedException e1) {
  					e1.printStackTrace();
  				}
  			}
  		} // End while
  		new Receiver(socket).start();
  	}
  
  	public void sendMsg(String msg) throws IOException {
  		Sender sender = null;
  		sender = new Sender(socket);
  		sender.setMsg(msg);
  		sender.start();
  	}
  	
  	public void start() throws Exception {
  		Scanner sc = new Scanner(System.in);
  		boolean sflag = true;
  		while (sflag) {
  			System.out.println("Input Msg.");
  			String str = sc.next();
  			sendMsg(str);
  			if(str.equals("q")) {
  				break;
  			}
  		}
  		sc.close();
  	}
  
  	// Inner Class
  	class Sender extends Thread {
  		Socket socket;
  		OutputStream out;
  		DataOutputStream dout;
  		String msg;
  
  		public Sender(Socket socket) throws IOException {
  			out = socket.getOutputStream();
  			dout = new DataOutputStream(out);
  		}
  		
  		public void setMsg(String msg) {
  			this.msg = msg;
  		}
  		
  		public void run() {
  			if(dout != null) {
  				try {
  					dout.writeUTF(msg);
  				} catch (IOException e) {
  					e.printStackTrace();
  				}
  			}
  		}
  	}
  
  	class Receiver extends Thread {
  		Socket socket;
  		InputStream in;
  		DataInputStream din;
  
  		public Receiver(Socket socket) throws IOException {
  			this.socket = socket;
  			in = socket.getInputStream();
  			din = new DataInputStream(in);
  		}
  
  		public void run() {
  			try {
  				while (rflag) {
  					String str = din.readUTF();
  					System.out.println(str);
  				}
  			} catch (Exception e) {
  
  			}
  		}
  	}
  
  	public static void main(String[] args) {
  		Client client = null;
  
  		try {
  			client = new Client("70.12.60.108", 8888);
  			client.start();
  		} catch (Exception e) {
  			e.printStackTrace();
  		}
  	}
  
  }
  
  
  // chat.jsp
  
  <%@ page language="java" contentType="text/html; charset=EUC-KR"
  	pageEncoding="EUC-KR"%>
  <!DOCTYPE html>
  <html>
  <head>
  <meta charset="EUC-KR">
  <title>Insert title here</title>
  <script
  	src="https://ajax.googleapis.com/ajax/libs/jquery/1.8.3/jquery.min.js"></script>
  <!-- <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script> -->
  <script>
  	function sendMsg(msg) {
  		$.ajax({
  			url : 'chat',
  			data : {
  				"msg" : msg
  			},
  			success : function(data) {
  				$('#msg').val('');
  			}
  		});
  	}
  
  	$(document).ready(function() {
  		$('#bt').click(function() {
  			var msg = $('#msg').val();/* msg 받아오기 */
  			sendMsg(msg);
  		});
  	});
  </script>
  </head>
  <body>
  	<h1>Chat Web Client</h1>
  
  	<input id="msg" type="text" name="msg">
  	<input id="bt" type="button" value="CHAT">
  
  </body>
  </html>
  
  // ChatServlet.java
  
  package chat;
  
  import java.io.IOException;
  import javax.servlet.ServletException;
  import javax.servlet.annotation.WebServlet;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  
  @WebServlet({ "/ChatServlet", "/chat" })
  public class ChatServlet extends HttpServlet {
  	private static final long serialVersionUID = 1L;
  
  	Client client;
  
  	public ChatServlet() {
  		try {
  			client = new Client("70.12.60.108", 8888);
  		} catch (IOException e) {
  			e.printStackTrace();
  		}
  	}
  
  	protected void service(HttpServletRequest request, HttpServletResponse response)
  			throws ServletException, IOException {
  		String msg = request.getParameter("msg");
  		client.sendMsg(msg);
  		
  	}
  
  }
  
  
  ```
  
  
  



* 1:n 채팅

  ㅇday05 

  ![server](https://user-images.githubusercontent.com/50862497/65924828-6f15c300-e429-11e9-9eae-368f9f1b0c8a.png)
  
  ```java
  //Server.java
  
  package tcp;
  
  import java.io.DataInputStream;
  import java.io.DataOutputStream;
  import java.io.IOException;
  import java.io.InputStream;
  import java.io.OutputStream;
  import java.net.ServerSocket;
  import java.net.Socket;
  import java.util.Collection;
  import java.util.HashMap;
  import java.util.Iterator;
  import java.util.Map;
  import java.util.Scanner;
  
  public class Server { // Server는 소켓만 만든다
  
  	boolean flag = true; // accept를 위한 while 루프 안에서 사용
  	boolean rflag = true;
  	Map<String, DataOutputStream> map = new HashMap<>(); // ip, port
  	Map<String, String> map2 = new HashMap<>(); // ip, id
  	Map<String, String> map3 = new HashMap<>(); // id, ip
  	
  	ServerSocket serverSocket;
  
  	public Server() {
  	}
  
  	public Server(int port) throws IOException {
  		serverSocket = new ServerSocket(port); // ServerSocket을 port(몇번)로 하겠다
  		System.out.println("Server Start..");
  		
  		Runnable r = new Runnable() {
  			@Override
  			public void run() {
  				try {
  					 while (flag) {
  						System.out.println("Server Ready..");
  						Socket socket = serverSocket.accept();
  						new Receiver(socket).start();
  						System.out.println(socket.getInetAddress());
  					} 
  				} catch (IOException e) {
  					e.printStackTrace();
  				}
  			}
  		};
  		new Thread(r).start();
  	}
  
  	public void start() throws IOException {
  		Scanner sc = new Scanner(System.in);
  		boolean sflag = true;
  		
  		while (sflag) {
  			System.out.println("Input Msg.");
  			String str = sc.next();
  			sendMsg(str);
  			
  			if (str.equals("q")) {
  				break;
  			}
  		}
  		
  		System.out.println("Bye....");
  		sc.close();
  	}
  
  	public void sendMsg(String msg) {
  		Sender sender = new Sender();
  		
  		sender.setType(false);
  		sender.setMsg(msg);
  		
  		sender.start();
  	}
  
  	public void sendMsgToTarget(String msg, String ip) {
  		Sender sender = new Sender();
  		
  		sender.setType(true);
  		sender.setMsg(msg);
  		sender.setTarget(ip);
  		
  		sender.start();
  	}
  	// Inner Class
  	class Sender extends Thread {
  		String msg;
  		boolean sendflag = true;
  		String targetIp;
  		
  		public void setMsg(String msg) {
  			this.msg = msg;
  		}
  		
  		public void setType(boolean type) {
  			this.sendflag = type;
  		}
  		
  		public void setTarget(String ip) {
  			this.targetIp = ip;
  		}
  		
  		// 전체-> msg~~~~
  		// else -> "false,target,msg"
  		public void run() {
  			if(sendflag) {
  				try {
  					DataOutputStream output = map.get(targetIp);
  					output.writeUTF(msg);
  				} catch (IOException e) {
  					// TODO Auto-generated catch block
  					e.printStackTrace();
  				}
  			}
  			
  			else {
  				Collection<DataOutputStream> col = map.values(); // values : key값 무시하고 value 꺼낼수있음
  				Iterator<DataOutputStream> it = col.iterator();
  			
  				while (it.hasNext()) {
  					try {
  						DataOutputStream output = it.next();
  						output.writeUTF(msg);
  					} catch (IOException e) {
  						e.printStackTrace();
  					}
  				}
  			}
  		}
  	}
  
  	class Receiver extends Thread {
  		Socket socket;
  		String ip;
  		
  		InputStream in;
  		DataInputStream din;
  		
  		OutputStream out;
  		DataOutputStream dout;
  		
  		public Receiver() {
  		}
  
  		public Receiver(Socket socket) throws IOException {
  			this.socket = socket;
  			
  			in = socket.getInputStream();
  			din = new DataInputStream(in);
  
  			out = socket.getOutputStream();
  			dout = new DataOutputStream(out);
  			
  			ip = socket.getInetAddress().toString();
  			map.put(ip, dout);
  			map2.put(ip, "unknown" + map.size());
  			map3.put("unknown" + map.size(), ip);
  			
  			System.out.println("접속자수:"+map.size());
  		}
  
  		public void run() {
  			try {
  				while (rflag) {
  					String str = din.readUTF();
  //					System.out.print(socket.getInetAddress()+": ");
  					
  					/*
  					 * if(socket.getInetAddress().toString().equals("/70.12.60.108")) {
  					 * System.out.print("지연 : "); }else
  					 * if(socket.getInetAddress().toString().equals("/70.12.60.106")) {
  					 * System.out.print("재영 : "); }else
  					 * if(socket.getInetAddress().toString().equals("/70.12.60.99")) {
  					 * System.out.print("지훈 : "); }else {
  					 * System.out.print(socket.getInetAddress()+": "); }
  					 */
  					
  					if(str.equals("q")) {
  						map.remove(ip);
  						System.out.println("Out");
  						System.out.println("접속자수:"+map.size());
  						break;
  					}
  					
  					String[] nic = str.split(" ");
  					
  					if(nic[0].equals("/닉네임")) {
  						if(map2.containsKey(socket.getInetAddress().toString())){
  							map2.replace(socket.getInetAddress().toString(),nic[1]);
  							map3.replace(nic[1], socket.getInetAddress().toString());
  						}
  					}
  					
  					else if(nic[0].equals("/귓속말")) {
  						// sendMsg2(ip, msg);
  						if(map2.containsValue(nic[1])){
  							String ip = map3.get(nic[1]);
  							sendMsgToTarget(map2.get(socket.getInetAddress().toString()) + " : (귓속말) " + nic[2], ip);
  						}
  					}
  					
  					else if(nic[0].equals("/리스트")) {
  						String ip = socket.getInetAddress().toString();
  						
  						String listMsg = "대화상대\n";
  						
  						Collection<String> nics = map2.values();
  						Iterator<String> it = nics.iterator();
  						
  						while(it.hasNext()) {
  							String nick = it.next();
  							listMsg += nick +"\n";
  						}
  						
  						sendMsgToTarget(listMsg,ip);
  					}
  					
  					else {
  						sendMsg(map2.get(socket.getInetAddress().toString()) + " : " + str);
  					}
  					
  					System.out.println(map2.get(socket.getInetAddress().toString()) + " : " + str);
  				}
  				
  				if(socket != null) {
  					socket.close();
  				}
  			} catch (Exception e) {
  				e.printStackTrace();
  			}
  		}
  	}
  
  	public static void main(String[] args) {
  		Server server = null;
  		
  		try {
  			server = new Server(8888);
  			server.start();
  		} catch (IOException e) {
  			e.printStackTrace();
  		}
  	}
  
  }
  
  //------------------------------------------------------------------------------------
  
  // Client.java
  
  package tcp;
  
  import java.io.DataInputStream;
  import java.io.DataOutputStream;
  import java.io.IOException;
  import java.io.InputStream;
  import java.io.OutputStream;
  import java.net.Socket;
  import java.util.Scanner;
  
  
  // socket만들고 reciver 받을준비, scanner로 입력받아 key in 하면 sender에 의해 메시지 보내기
  public class Client {
  
  	Socket socket;
  	boolean rflag = true;
  
  	public Client() {
  
  	}
  
  	public Client(String ip, int port) throws IOException {
  		boolean flag = true;
  		while (flag) {
  			try {
  				socket = new Socket(ip, port);
  				if (socket != null && socket.isConnected()) {
  					break;
  				}
  			} catch (Exception e) { // 서버가 안켜져있으면
  				System.out.println("Re-Try");
  				try {
  					Thread.sleep(3000);
  				} catch (InterruptedException e1) {
  					e1.printStackTrace();
  				}
  			}
  		} // End while
  		new Receiver(socket).start();
  	}
  
  	public void sendMsg(String msg) throws IOException {
  		Sender sender = null;
  		sender = new Sender(socket);
  		sender.setMsg(msg);
  		sender.start();
  	}
  	
  	public void start() throws Exception {
  		Scanner sc = new Scanner(System.in);
  		boolean sflag = true;
  		while (sflag) {
  			System.out.println("Input Msg.");
  			String str = sc.next();
  			sendMsg(str);
  			if(str.equals("q")) {
  				break;
  			}
  		}
  		sc.close();
  	}
  
  	// Inner Class
  	class Sender extends Thread {
  		Socket socket;
  		OutputStream out;
  		DataOutputStream dout;
  		String msg;
  
  		public Sender(Socket socket) throws IOException {
  //			this.socket = socket;
  			out = socket.getOutputStream();
  			dout = new DataOutputStream(out);
  		}
  		
  		public void setMsg(String msg) {
  			this.msg = msg;
  		}
  		
  		public void run() {
  			if(dout != null) {
  				try {
  					dout.writeUTF(msg);
  				} catch (IOException e) {
  					e.printStackTrace();
  				}
  			}
  		}
  	}
  
  	class Receiver extends Thread {
  		Socket socket;
  		InputStream in;
  		DataInputStream din;
  
  		public Receiver(Socket socket) throws IOException {
  			this.socket = socket;
  			in = socket.getInputStream();
  			din = new DataInputStream(in);
  		}
  
  		public void run() {
  			
  			try {
  				while (rflag) {
  					String str = din.readUTF();
  					System.out.println(str);
  				}
  			} catch (Exception e) {
  
  			}
  		}
  	}
  
  	public static void main(String[] args) {
  		Client client = null;
  
  		try {
  			client = new Client("70.12.60.110", 8888);
  			client.start();
  		} catch (Exception e) {
  			e.printStackTrace();
  		}
  	}
  
  }
  
  //------------------------------------------------------------------------------------
  
  // MainActivity.java
  
  package com.example.webserversocketexcersice;
  
  import androidx.appcompat.app.AppCompatActivity;
  
  import android.os.AsyncTask;
  import android.os.Bundle;
  import android.text.method.ScrollingMovementMethod;
  import android.util.Log;
  import android.view.View;
  import android.widget.EditText;
  import android.widget.TextView;
  
  import java.io.DataInputStream;
  import java.io.DataOutputStream;
  import java.io.IOException;
  import java.io.InputStream;
  import java.io.OutputStream;
  import java.net.Socket;
  
  public class MainActivity extends AppCompatActivity {
  
      Socket socket = null;
      OutputStream out;
      DataOutputStream dout;
      ReceiveThread receiveThread;
  
      TextView textView;
      EditText editText;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
          textView = findViewById(R.id.textView);
          editText = findViewById(R.id.msg);
          textView.setVerticalScrollBarEnabled(true);
          textView.setMovementMethod(new ScrollingMovementMethod());
          receiveThread = new ReceiveThread("70.12.60.106",8888);
  
          if(receiveThread!=null){
              receiveThread.execute();
          }
      }
  
      private void scrollBottom(TextView textView) {
          int lineTop =  textView.getLayout().getLineTop(textView.getLineCount()) ;
          int scrollY = lineTop - textView.getHeight();
          if (scrollY > 0) {
              textView.scrollTo(0, scrollY);
          } else {
              textView.scrollTo(0, 0);
          }
      }
  
  
      @Override
      protected void onDestroy() {
          sendProgress("q");
          super.onDestroy();
      }
      void sendProgress(String text){
          final String msg = text;
          if(msg.equals("q")){
              receiveThread.cancel(true);
          }
          final Thread sendThread = new Thread(new Runnable() {
              @Override
              public void run() {
                  try {
                      out = socket.getOutputStream();
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
                  dout = new DataOutputStream(out);
                  if (dout != null) {
                      try {
                          dout.writeUTF(msg);
  
                      } catch (IOException e) {
                          e.printStackTrace();
                      }
                  }
              }
          });
          sendThread.start();
          editText.setText("");
      }
      public void SendClick(View v) throws IOException {
          sendProgress(editText.getText().toString());
      }
      class ReceiveThread extends AsyncTask<String,Object,Integer>{
  
          Socket socket1;
          InputStream in;
          DataInputStream din;
          String ip;
          int port;
  
          public ReceiveThread(String ip, int port) {
              this.ip = ip;
              this.port = port;
          }
  
          @Override
          protected void onProgressUpdate(Object... values) {
              String cmd = (String)values[0];
              Log.d("cmd",cmd);
              if(cmd.equals("socket")){
                  socket = (Socket)values[1];
              }
              else {
                  textView.append("\n"+(String)values[1]);
                  scrollBottom(textView);
              }
          }
  
          @Override
          protected Integer doInBackground(String... strings) {
              try {
                  socket1 = new Socket(ip,port);
                  socket = socket1;
                  in = socket1.getInputStream();
                  Object[] container = {new String("socket"),socket1};
                  publishProgress(container);
                  din = new DataInputStream(in);
              } catch (IOException e) {
                  e.printStackTrace();
              }
              try {
                  while (true) {
                      final String str = din.readUTF();
                      Log.d("deceive", str);
                      System.out.println("receice thread is running");
                      publishProgress(new String("message"),str);
                  }
              } catch (IOException e) {
                  e.printStackTrace();
              }
              return null;
          }
      }
  
  }

  ```
  
  

