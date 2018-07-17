---
title: NIO的使用
categories:
  - 笔记
tags:
  - NIO 
  - Java基础
  - socket
date: 2018-07-14 15:07:48
---
 这是摘要
 <!-- more -->

**同步和异步说的是消息的通知机制,阻塞和非阻塞说的是线程的状态,**
1. 同步阻塞io
指的就是传统的Java IO,线程发起io后会阻塞,等到io结束之后才会恢复
2. 同步非阻塞io(消息机制会阻塞,io不会阻塞)
Java中的NIO(Non-blocking IO)是这种模型,线程发起io后可以去干其他事情,但是另外起一个线程去监听或者轮询io状态(是否有可读数据,或者是否可以写入)
```java 
//创建Selector监听IO事件,
Selector selector = Selector.open();  

//for循环中,为每一个监听的port打开一个channel,并设置成非阻塞
ServerSocketChannel ssc = ServerSocketChannel.open();
ssc.configureBlocking( false );
 
ServerSocket ss = ssc.socket();
InetSocketAddress address = new InetSocketAddress( ports[i] );
ss.bind( address );

SelectionKey key = ssc.register( selector, SelectionKey.OP_ACCEPT );//将channel注册到selector

//while循环
while(ture){
	//该方法会阻塞,直到至少有一个已经注册的事件发生时,会返回发生事件的数量
	int num = selector.select();
 	//事件发生后就可以获得SelectionKey集合,然后进行迭代处理
	Set selectedKeys = selector.selectedKeys();
	Iterator it = selectedKeys.iterator();
	 
	while (it.hasNext()) {
	     SelectionKey key = (SelectionKey)it.next();
	     // ... deal with I/O event ...
	     if ((key.readyOps() & SelectionKey.OP_ACCEPT)
                        == SelectionKey.OP_ACCEPT) {
            // Accept the new connection
         } else if ((key.readyOps() & SelectionKey.OP_READ)
                == SelectionKey.OP_READ) {
            // Read the data
		 }
	}
}
```

3. 异步非阻塞io
Java中的AIO((Asynchronous IO))是这种模型,线程发起io后可以直接干其他事前,系统会监听io状态,IO完成后会通知线程

### 使用
参考地址:https://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html
1. NIO
```java
public class FastCopyFile {
    static public void main(String args[]) throws Exception {
        if (args.length < 2) {
            System.err.println("Usage: java FastCopyFile infile outfile");
            System.exit(1);
        }

        String infile = args[0];
        String outfile = args[1];

        FileInputStream fin = new FileInputStream(infile);
        FileOutputStream fout = new FileOutputStream(outfile);

        FileChannel fcin = fin.getChannel();
        FileChannel fcout = fout.getChannel();

        ByteBuffer buffer = ByteBuffer.allocateDirect(1024);

        while (true) {
            //设置缓存区,使之可以读入新数据
            //position = 0;limit = capacity;mark = -1;
            buffer.clear();
            //将输入通道中的数据读到缓存区
            int r = fcin.read(buffer);
            //判断输入通道的数据是不是读完了
            if (r == -1) {
                break;
            }
            //设置缓存区,然缓存区中的数据可以写入新的通道
            //limit = position;position = 0;mark = -1;
            buffer.flip();
            //将数据从缓存区写入输出通道
            fcout.write(buffer);
        }
    }
}
```