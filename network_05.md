# TCP/IP

09.20

TCP/IP :  transmission control protocol/internet protocol

소켓 : 프로세스간의 통신에 사용되는 양쪽 끝단

프로세스간의 통신을 위해서는 소켓이 필요

Exception 처리 잘하기



Http protocol : 브라우저와 웹서버의 규약

server socket 열어두기 / socket / outputStream



재접속을 위해 throw안던짐



* tcp1 package

  ```java
  // Server.java
  
  package tcp1;
  
  import java.io.BufferedWriter;
  import java.io.IOException;
  import java.io.OutputStream;
  import java.io.OutputStreamWriter;
  import java.net.ServerSocket;
  import java.net.Socket;
  
  public class Server {
  
  	int port;
  	ServerSocket serverSocket;
  	Socket socket;
  
  	OutputStream out;
  	OutputStreamWriter osw;
  	BufferedWriter bw;
  	
  	boolean flag = true;
  
  	public Server(int port) throws IOException {
  		this.port = port;
  		serverSocket = new ServerSocket(port);
  	}
  
  	public void startServer() throws IOException {
  		System.out.println("Server Start..");
  		while (flag) {
  			try {
  				System.out.println("Server Ready.."); // 여기서 thread가 멈춘다
  				// client가 server에 접속하면
  				socket = serverSocket.accept(); // socket 만들기
  				System.out.println("Accepted.." + socket.getInetAddress()); // 누가 들어왔는지
  				
  				out = socket.getOutputStream();
  				osw = new OutputStreamWriter(out);
  				bw = new BufferedWriter(osw);
  				bw.write("안녕^-^");
  				bw.flush();
  				
  			} catch (IOException e) {
  				throw e;
  			} finally {
  				if (bw != null) {
  					bw.close();
  				}				
  				if (socket != null) {
  					socket.close();
  				}
  			}
  		} // end while
  		System.out.println("Server End..");
  	}
  
  	public static void main(String[] args) {
  		Server server = null;
  		try {
  			server = new Server(8888);
  			server.startServer();
  		} catch (IOException e) {
  			e.printStackTrace();
  		}
  	}
  
  }
  
  //------------------------------------------------------------------------------------
  
  // Clinet.java
  
  package tcp1;
  
  import java.io.BufferedReader;
  import java.io.IOException;
  import java.io.InputStream;
  import java.io.InputStreamReader;
  import java.net.Socket;
  
  public class Client {
  	
  	String ip;
  	int port;
  	Socket socket;
  		
  	InputStream in;
  	InputStreamReader inr;
  	BufferedReader br;
  	
  	public Client(String ip, int port) { // port, ip 받기
  		this.ip = ip;
  		this.port = port;
  	}
  	
  	// 객체만들고 함수만들어서 보내기
  	public void connect() {
  		try {
  			socket = new Socket(ip, port); //serverSocket과 연결
  		} catch (Exception e) { // server가 켜질때까지 계속 소켓을 만든다
  			boolean flag = true;
  			while(flag) {
  				try {
  					Thread.sleep(2000); // 2초마다
  					socket = new Socket(ip, port);
  					break; // socket이 만들어지면
  				} catch (Exception e1) {
  					System.out.println("retry...");
  				}
  			}
  		} 
  	}
  	
  	public void request() throws Exception{ // socket이 만들어져서 가져올 수 있음
  		try {
  			in = socket.getInputStream();
  			inr = new InputStreamReader(in);
  			br = new BufferedReader(inr);
  			String str = br.readLine();
  			System.out.println(str);
  		} catch (IOException e) {
  			e.printStackTrace();
  		} finally {
  			if(in != null) {
  				in.close();
  			}
  			if(inr != null) {
  				inr.close();
  			}
  			if(br != null) {
  				br.close();
  			}
  			if(socket != null) { // Stream과 socket은 별개
  				socket.close();
  			}
  		}
  	}
  	public static void main(String[] args) {
  		Client client = null;
  		client = new Client("70.12.60.106", 8888); // 여기로 접속하겠다
  		client.connect(); // 연결
  		try {
  			client.request();
  		} catch (Exception e) {
  			e.printStackTrace();
  		}
  	}
  
  }
  
  ```

  



----------------------------



server inputStream

client가 메시지갖고 들어가서 server는 한번 찍어주고 응답

```java
// Server.java
```





