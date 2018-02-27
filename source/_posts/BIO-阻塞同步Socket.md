---
title: BIO-阻塞同步Socket
date: 2018-02-27 14:13:35
tags: Netty
---

### Server
```
package bhz.bio;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;


public class Server {

	final static int PROT = 8765;
	
	public static void main(String[] args) {
		
		ServerSocket server = null;
		try {
			server = new ServerSocket(PROT);
			System.out.println(" server start .. ");
			//进行阻塞
			Socket socket = server.accept();
			//新建一个线程执行客户端的任务
			new Thread(new ServerHandler(socket)).start();
			
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if(server != null){
				try {
					server.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			server = null;
		}
	}	
}

//ServerHandler
package bhz.bio;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;

public class ServerHandler implements Runnable{

	private Socket socket ;
	
	public ServerHandler(Socket socket){
		this.socket = socket;
	}
	
	@Override
	public void run() {
		BufferedReader in = null;
		PrintWriter out = null;
		try {
			in = new BufferedReader(new InputStreamReader(this.socket.getInputStream()));
			out = new PrintWriter(this.socket.getOutputStream(), true);
			String body = null;
			while(true){
				body = in.readLine();
				if(body == null) break;
				System.out.println("Server :" + body);
				out.println("服务器端回送响的应数据.");
			}
			
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if(in != null){
				try {
					in.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			if(out != null){
				try {
					out.close();
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
			if(socket != null){
				try {
					socket.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			socket = null;
		}
	}
}

```
### Client

```
package bhz.bio;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;

public class Client {

	final static String ADDRESS = "127.0.0.1";
	final static int PORT = 8765;
	
	public static void main(String[] args) {
		
		Socket socket = null;
		BufferedReader in = null;
		PrintWriter out = null;
		
		try {
			socket = new Socket(ADDRESS, PORT);
			in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
			out = new PrintWriter(socket.getOutputStream(), true);
			
			//向服务器端发送数据
			out.println("接收到客户端的请求数据...");
			String response = in.readLine();
			System.out.println("Client: " + response);
			
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if(in != null){
				try {
					in.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			if(out != null){
				try {
					out.close();
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
			if(socket != null){
				try {
					socket.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			socket = null;
		}
	}
}

```

