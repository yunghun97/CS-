# 💾 Zero Copy

## Zero Copy란?
컴퓨터 시스템에서 CPU의 개입을 받지 않고 한 메모리의 영역에서 다른 메모리의 영역으로 데이터를 카피하는 작업을 말한다. 이는 CPU 사이클을 절약하여서 네트워크나 SSD와 같이 빠른 데이터가 필요한 시스템에서 유용하게 이용된다.
  
## 😅 일반적인 파일 전송
- 파일 서버나 정적 파일을 서비스하는 웹 애플리케이션은 디스크에서 파일 컨텐츠를 읽어 네트워크 소켓으로 데이터를 전송하는 일을 반복적으로 수행한다.
- 이 동작에 운영체제 내부에서의 불필요한 컨텍스트 스위칭, 데이터 복사가 수반된다

일반적인 소스코드
```java
byte[] applicationBuffer = new byte[1024];
read(fildFd, applicationBuffer, len);
send(socketFd, applicationBuffer, len);

// 버퍼를 할당 받고 디스크에서 파일을 버퍼로 읽어 들여서 다시 소켓으로 전달하는 형태 
// while 문을 사용해 더 큰 데이터를 전송할 수 있지만 기본적으로 read() send() 시스템 콜이 반복되는 형태이다.
```
![image](https://user-images.githubusercontent.com/71022555/164708479-0aa96e3e-c692-4078-a009-e5200ee0ba7b.png)  
일반적인 상황에서의 파일 전송(출처 : IBM 문서)

### 실행 로직
1. 유저가 read() 시스템 콜을 호출해 파일을 읽어달라고 요청하면, DMA 엔진에 의해서 디스크에 존재하는 파일의 내용이 커널 주소공간에 위치한 Read Buffer에 복사된다.
2. 커널 주소공간에 위치한 Read buffer는 사용자가 접근할 수 없기 때문에 사용자가 read() 함수 호출시 파라미터로 전달한 Application buffer에 Read buffer의 내용을 복사해준다. 복사 완료 시 함수 호출에서 return 한다.
3. 사용자는 Application buffer로 읽어 들인 데이터를 소켓으로 전송하기 위해 send() 함수를 호출한다. send() 함수 호출 시 Application buffer를 파라미터로 전달하면 커널 영역에 위치한 Socket buffer로 데이터를 복사한다.
4. Socket buffer에 있는 데이터는 실제 네트워크로 전송하기 위해 네트워크 장비(NIC)에 있는 buffer로 다시 복사도니다.
  
- 이렇게 4개의 버퍼에 동일한 데이터가 복사되는 이 과정은 애플리케이션의 CPU 자원을 소모한다. 이런 버퍼링은 메모리와 디스크, 메모리와 네트워크 장비 사이의 속도 차이를 만회하고자 만들어진 성능 개선 장치이다.  
- 예를 들어 1KB 정도의 작은 데이터를 파일에서 읽으면 커널은 1KB 이후 데이터까지 한번에 읽어서 페이지 캐시에 로드한다. 이 후, 그 다음 데이터 요청 시 물리적인 Input/Output을 수행하지 않고 페이지 캐시에서 읽어서 사용자에게 준다. 데이터를 미리 읽어서 (Read Ahead) 성능 개선을 도모한 것  
- 이런 버퍼링이 성능 저하를 유발하는 병목으로 작용할 수 있다. 사용자가 요청하는 데이터의 크기가 커널이 유지하는 버퍼의 사이즈보다 큰 경우 미리 읽는(Read Ahead) 성능 개선 효과보다 여러 단계에 걸쳐 데이터를 복사하는 비효율이 더 커지게 된다.
  
## 😉 Zero Copy
리눅스 2.2 버전
```c
#include  <sys/sendfile.h>
ssize_t endfile(int out_fd, int in_fd, off_t * offset ",size_t" " count" );
```
Java (nio 패키지에 transferTo(), transferFrom() 메소드로 구현되어있다.)
```java
public void transferTo(long position, long count, WritableByteChannel target);

// transferTo(), transferFrom() 메소드 역시 sendfile() 시스템 콜을 이용해 구현되었다. 

toChannel.transferTo(position, to, toChannel);
```
![image](https://user-images.githubusercontent.com/71022555/164710687-697c868f-bb7b-41cc-a6b1-44a9a244ac0e.png)  
transferTo() 메소드를 이용한 파일전송(출처 : IBM 문서)
  
### 실행 로직
1. 사용자가 transferTo() 메소드를 이용해 파일 전송을 요청한다. read()와 send() 함수가 하나로 합쳐진 형태의 시스템 콜이다. read() 시스템 콜과 마찬가지로 DMA 엔진이 디스크에서 파일을 읽어 커널 주소 공간에 위치한 Read buffer로 데이터를 복사한다. 
2. 커널 모드에서 유저 모드로 컨텍스트 스위칭하지 않고 바로 Socket buffer로 데이터를 복사한다. 
3. Socket buffer에 복사된 데이터를 DMA 엔진을 통해 NIC buffer로 복사되어 진다. 
4. 데이터가 전송되고 transferTo() 메소드에서 리턴한다. 
  
컨텍스트 스위칭이 4번에서 2번으로 줄었다. transferTo() 메소드 호출시 커널 모드로 한번, 종료시 유저모드로 한번 총 2번의 컨텍스트 스위칭이 발생한다.  
동작들이 유저 주소공간에 있는 Application Buffer로 복사되어지지 않기 때문에 데이터의 복사본이 4군데에서 3군데로 줄어들어다. 따라서 데이터 복사 동작도 한 번 줄어들었다.
  
## 😀 더 개선된 Zero Copy
리눅스 커널 2.4 이후 버전에서는 NIC 장비가 "Gather Operation"을 지원할 경우 복사본을 더 줄일 수 있게 되었다.  
![image](https://user-images.githubusercontent.com/71022555/164711166-90c403ed-82a3-4199-b5bd-8ca58213be7f.png)  
NIC의 도움을 받은 제로카피가 적용된 transferTo() 메소드(출처 : IBM 문서)
  
### 실행 로직
커널 내부 동작이 좀 더 개선된 것
1. 사용자가 transferTo() 메소드를 호출한다. DMA 엔진이 디스크에서 파일을 읽어 커널에 위치한 Read buffer로 데이터를 복사한다. 여기까지는 동일하다. 
2. 데이터가 소켓 버퍼로 복사되어지지는 않는다. 대신 데이터가 저장된 위치와 데이터 사이즈에 대한 정보와 함께 디스크립터(descriptor)가 소켓 버퍼에 추가된다. DMA 엔진은 이 정보를 이용해 Read buffer에 있는 데이터를 NIC 버퍼에 바로 복사하고, 네트워크로 데이터를 전송한다.  

#### 이런 추가적인 최적화는 NIC의 gather operation과 프로토콜의 checksum 기능이 필요하다. TCP와 UDP의 경우 Checksum 기능이 구현되어 있다. NIC 장비의 "Gather operation" 지원의 이유는 stackoverflow에서 힌트를 얻을 수 있다. 
  
## 😊 성능상 이점
|File size|Normal file transfer(ms)|tarnsferTo(ms)|
|:---:|:---:|:---:|
|7MB|156|45|
|21MB|337|128|
|63MB|843|387|
|98MB|1320|617|
|200MB|2124|1150|
|350MB|3631|1762|
|700MB|13498|4422|
|1GB|18399|8537|

## 😁 요약
#### 디스크에서 파일을 읽은 후 별다른 처리 없이 바로 네트워크로 전송하는 파일 서버나 정적 파일을 전송하는 웹 서버의 경우 제로 카피를 사용하면 성능 개선 효과를 얻을 수 있다. 특히 네트워크 속도가 매우 빨라서 서버의 성능상 병목점(bottleneck)이 CPU로 몰릴 수록 불필요한 데이터 복사를 제거해 성능 개선을 얻을 수 있다. 
  
## 소스
### 일반적인 파일 전송
클라이언트
```java
import java.io.DataOutputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.net.Socket;
import java.net.UnknownHostException;

public class TraditionalClient {
    
    
    
    public static void main(String[] args) {

	int port = 2000;
	String server = "localhost";
	Socket socket = null;
	String lineToBeSent;
	
	DataOutputStream output = null;
	FileInputStream inputStream = null;
	int ERROR = 1;
	
	
	// connect to server
	try {
	    socket = new Socket(server, port);
	    System.out.println("Connected with server " +
				   socket.getInetAddress() +
				   ":" + socket.getPort());
	}
	catch (UnknownHostException e) {
	    System.out.println(e);
	    System.exit(ERROR);
	}
	catch (IOException e) {
	    System.out.println(e);
	    System.exit(ERROR);
	}
	
	try {
		String fname = "sendfile/NetworkInterfaces.c";
		inputStream = new FileInputStream(fname);
		
	    output = new DataOutputStream(socket.getOutputStream());
	    long start = System.currentTimeMillis();	    
	    byte[] b = new byte[4096];
	    long read = 0, total = 0;
	    while((read = inputStream.read(b))>=0) {
		total = total + read;
	    	output.write(b);
	    }
	    System.out.println("bytes send--"+total+" and totaltime--"+(System.currentTimeMillis() - start));
	}
	catch (IOException e) {
	    System.out.println(e);
	}

	try {
		output.close();
	    socket.close();
	    inputStream.close();
	}
	catch (IOException e) {
	    System.out.println(e);
	}
    }    
}
```
서버
```java
import java.net.*;
import java.io.*;

public class TraditionalServer {
    
    public static void main(String args[]) {
	
	int port = 2000;
	ServerSocket server_socket;
	DataInputStream input;
	
	try {
	    
	    server_socket = new ServerSocket(port);
	    System.out.println("Server waiting for client on port " + 
			       server_socket.getLocalPort());
	    
	    // server infinite loop
	    while(true) {
		Socket socket = server_socket.accept();
		System.out.println("New connection accepted " +
				   socket.getInetAddress() +
				   ":" + socket.getPort());
		input = new DataInputStream(socket.getInputStream()); 
		// print received data 
		try {
			byte[] byteArray = new byte[4096];
		    while(true) {
		    	int nread = input.read(byteArray , 0, 4096);
		    	if (0==nread) 
		    		break;
		    }
		}
		catch (IOException e) {
		    System.out.println(e);
		}
		
		// connection closed by client
		try {
		    socket.close();
		    System.out.println("Connection closed by client");
		}
		catch (IOException e) {
		    System.out.println(e);
		}
		
	    }
	    
	    
	}
	
	catch (IOException e) {
	    System.out.println(e);
	}
    }
}
```
### Zero Copy
클라이언트
```java
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.SocketAddress;
import java.nio.channels.FileChannel;
import java.nio.channels.SocketChannel;

public class TransferToClient {
	
	public static void main(String[] args) throws IOException{
		TransferToClient sfc = new TransferToClient();
		sfc.testSendfile();
	}
	public void testSendfile() throws IOException {
	    String host = "localhost";
	    int port = 9026;
	    SocketAddress sad = new InetSocketAddress(host, port);
	    SocketChannel sc = SocketChannel.open();
	    sc.connect(sad);
	    sc.configureBlocking(true);

	    String fname = "sendfile/NetworkInterfaces.c";
	    long fsize = 183678375L, sendzise = 4094;
	    
	    // FileProposerExample.stuffFile(fname, fsize);
	    FileChannel fc = new FileInputStream(fname).getChannel();
            long start = System.currentTimeMillis();
	    long nsent = 0, curnset = 0;
	    curnset =  fc.transferTo(0, fsize, sc);
	    System.out.println("total bytes transferred--"+curnset+" and time taken in MS--"+(System.currentTimeMillis() - start));
	    //fc.close();
	  }
}
```
서버
```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.nio.ByteBuffer;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;


public class TransferToServer  {
  ServerSocketChannel listener = null;
  protected void mySetup()
  {
    InetSocketAddress listenAddr =  new InetSocketAddress(9026);

    try {
      listener = ServerSocketChannel.open();
      ServerSocket ss = listener.socket();
      ss.setReuseAddress(true);
      ss.bind(listenAddr);
      System.out.println("Listening on port : "+ listenAddr.toString());
    } catch (IOException e) {
      System.out.println("Failed to bind, is port : "+ listenAddr.toString()
          + " already in use ? Error Msg : "+e.getMessage());
      e.printStackTrace();
    }

  }

  public static void main(String[] args)
  {
    TransferToServer dns = new TransferToServer();
    dns.mySetup();
    dns.readData();
  }

  private void readData()  {
	  ByteBuffer dst = ByteBuffer.allocate(4096);
	  try {
		  while(true) {
			  SocketChannel conn = listener.accept();
			  System.out.println("Accepted : "+conn);
			  conn.configureBlocking(true);
			  int nread = 0;
			  while (nread != -1)  {
				  try {
					  nread = conn.read(dst);
				  } catch (IOException e) {
					  e.printStackTrace();
					  nread = -1;
				  }
				  dst.rewind();
			  }
		  }
	  } catch (IOException e) {
		  e.printStackTrace();
	  }
  }
}
```
---
참고 자료  
https://free-strings.blogspot.com/2016/04/zero-copy.html
https://soft.plusblog.co.kr/7
https://gist.github.com/freestrings