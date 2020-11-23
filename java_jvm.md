##jvm的运行机制
JVM是运行Java字节码文件的虚拟机，包括一套字节码指令集、一组程序寄存器、虚拟机栈、虚拟机堆、一个方法区和垃圾回收器。

    具体运行活成如下：

    1）java源文件被编译为字节码文件。

    2）JVM将字节码文件编译为相应操作系统的机器码。

    3）机器码调用相应操作系统的本地方法库执行相应的方法。


多线程 jvm后台运行线程主要有以下几个

    1.虚拟机线程
	2.周期性任务线程
	3.GC线程
	4.编译器线程
	5.信号分发线程
JVM的内存区域

    内存区域分为：线程私有域（程序计数器、虚拟机栈、本地方法区）、线程共享区域（堆和方法区）和直接内存。
	
	程序计数器属于线程私有的内存区域，它是唯一没有内存溢出的区域。

	虚拟机栈：线程私有，java方法的执行过程

	本地方法区：线程私有，与虚拟机栈的作用类似，本地方法区为Native方法服务。

	堆：是被线程共享的内存区域，也是垃圾回收器进行垃圾回收的主要内存区域。

	方法区：用于存储常量、静态变量、类信息、运行时常量池的数据
###垃圾回收与算法

	Java采用引用计数法和可达性分析来确定对象是否应该被回收，其中引用计数法容易产生循环引用
	的问题，可达性分析通过搜索算法来实现。
	
1.引用计数法
	
	如果想要操作一个，必须获取对象的引用，因此通过引用计数法来判断一个对象是否可以被回收。对象添加一个引用时，

	引用计数加1；对象删除引用，引用计数减1；如果对象引用计数为0，则对象没有被引用，可以回收。

	引用计数法容易产生循环引用问题，循环引用指两个对象互相引用，导致他们的引用一直存在，而不能被回收。
2.可达性分析

	定义一些GC Roots对象作为起点，向下搜索，如果GC Roots和对象之间没有可达路径，则称该对象是不可达的。不可达的对象要经过至少两次标记才能判定是否能被回收，如果两次标记后该对象仍然是不可达的，则将被垃圾回收器回收。
###垃圾税收算法
	Java常用的垃圾回收算法有标记清除法、复制算法、标记整理法、分代收集。
1.标记清除法
	
	其过程分两个阶段，在标记阶段标记所有需要回收的对象，在清除阶段清除可回收的对象并释放其占用的内

	存空间，此算法在清理对象所占用的空间后没有重新整理内存空间。
2.复制算法

	复制算法首先将内存划分为两块大小相等的内存空间 1和 2，1区域的对象存储满后，对1的对象进行一次标

	记，将标记后存活的对象复制到2区域，这个时候区域1 不存在任何存活的对象，直接清理，此算法将可用内

	存压缩到原来一半，浪费大量内存
3.标记整理算法
	
	标记完成后，将活的对象移到内存的另一端，然后清除该端的对象并释放内存。
4.分代收集算法
	
	针对不同的对象类，采用不同的垃圾回收算法，因此叫分代收集算法。
###Java中的四种引用类型
	分别是强引用、软引用、弱引用和虚引用。
###Java网络编程模型
> 1.阻塞I/O模型

	在读写数据时客户端发生阻塞。例如：data=socket.read();
> 2.非阻塞I/O模型

	此模型是指用户线程发起一个I/O操作后，无需阻塞可以马上得到内核返回的结果，如果内核返回结果为false,表示
	
	内核数据还没好，需要稍后再发起I/O操作，一旦内核数据准备好了，并且再次受到用户线程的请求，内核会立即将结

	果返回给用户线程。
> 3.异步I/O模型

	此模型中，用户线程会发起一个Asynchronous read操作到内核，内核在街道Asynchronous read请求一个后会

	立刻返回状态，来说明请求是否发起成功，此过程用户线程不会发生任何阻塞。接着内核会等待数据准备完成，并将

	数据复制到用户的线程中，通知用户异步读操作已完成。
> 4.java NIO

	Java NIO主要涉及三大核心内容：Selector(选择器)、Channel(通道)和Buffer(缓冲区)

	1)I/O是面向流的，NIO是面向缓冲区的，在面向流的操作中，数据只能在流中进行读写，数据没有缓冲，因此字节流

	无法前后移动，NIO每次都是从一个Channel读取到Buffer中，再冲Buffer写入Channel中，方便在缓冲区进行数据前后移动等操作。
	2)传统I/O的流操作是阻塞模式，NIO的流操作是非阻塞模式。
	
Java NIO的使用:

	服务端

	```
	public class MyServer {
	    private int size=1024;
	    private ServerSocketChannel serverSocketChannel;
	    private ByteBuffer byteBuffer;
	    private Selector selector;
	    private int remoteClientNum=0;
	    public MyServer(int port){
	        //在构造函数中初始化监听
	        try {
	            initChannel(port);
	        } catch (IOException e) {
	            e.printStackTrace();
	            System.exit(-1);
	        }
	    }
	    //Channel的初始化
	    public void initChannel(int port) throws IOException {
	        //打开Channel
	        serverSocketChannel=ServerSocketChannel.open();
	        //设置为非阻塞模式
	        serverSocketChannel.configureBlocking(false);
	        //绑定端口
	        serverSocketChannel.bind(new InetSocketAddress(port));
	        System.out.println("listenter on port:"+port);
	        //创建选择器
	        selector=Selector.open();
	        //向选择器注册通道
	        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
	        //分配缓冲区的大小
	        byteBuffer=ByteBuffer.allocate(size);
	    }
	    //监听器，用于监听Channel上的数据变化
	    public void listener() throws IOException {
	        while (true){
	            //返回的int值表示有多少个Channel处于就绪状态
	            int n=selector.select();
	            if(n==0){
	                continue;
	            }
	            //每个selector对应多个SelectionKey,每个SelectionKey一个Channel
	            Iterator<SelectionKey> iterator=selector.selectedKeys().iterator();
	            while (iterator.hasNext()){
	                SelectionKey key=iterator.next();
	                //如果SelectionKey处于连接就绪状态，则开始接受客户端的连接
	                if(key.isAcceptable()){
	                    //获取Channel
	                    ServerSocketChannel server=(ServerSocketChannel)key.channel();
	                    //Channel接受连接
	                    SocketChannel channel=server.accept();
	                    //Channel注册
	                    registerChannel(selector,channel,SelectionKey.OP_READ);
	                    //远程客户端的连接数统计
	                    remoteClientNum++;
	                    System.out.println("online client num="+remoteClientNum);
	                    write(channel,"hello client".getBytes());
	                    //如果通道处于就绪装状态，则读取通道的数据
	                    System.out.println(key.isReadable());
	                    System.out.println(key.isAcceptable());
	                    if(key.isReadable()){
	                        read(key);
	                    }
	                    iterator.remove();
	                }
	            }
	        }
	    }
	
	    private void read(SelectionKey key) throws IOException {
	        System.out.println("+++++++++++++");
	        SocketChannel socketChannel=(SocketChannel)key.channel();
	        int count;
	        byteBuffer.clear();
	        //从通道读数据到缓存区
	        while ((count=socketChannel.read(byteBuffer))>0){
	            //byteBuffer写模式变为读模式
	            byteBuffer.flip();
	            while (byteBuffer.hasRemaining()){
	                System.out.println("---------------------");
	                System.out.println((char) byteBuffer.get());
	            }
	            byteBuffer.clear();
	        }
	        if(count<0){
	            socketChannel.close();
	        }
	    }
	
	
	    public void write(SocketChannel channel,byte[] writeData) throws IOException {
	        byteBuffer.clear();
	        byteBuffer.put(writeData);
	        //从写模式变为读模式
	        byteBuffer.flip();
	        //将缓存区的数据写入通道
	        channel.write(byteBuffer);
	    }
	
	    public void registerChannel(Selector selector,SocketChannel channel,int opRead) throws IOException {
	        if (channel==null){
	            return;
	        }
	        channel.configureBlocking(false);
	        channel.register(selector,opRead);
	    }
	}

客户端

	```
		public class MyClient {
	    private int size=1024;
	    private ByteBuffer byteBuffer;
	    private SocketChannel socketChannel;
	    public void connectServer() throws IOException {
	//        socketChannel.open();
	        socketChannel=SocketChannel.open();
	        socketChannel.connect(new InetSocketAddress("127.0.0.1",9999));
	        socketChannel.configureBlocking(false);
	        byteBuffer=ByteBuffer.allocate(size);
	        receive();
	    }
	
	
	    public void receive() throws IOException {
	        while (true){
	            byteBuffer.clear();
	            int count;
	            //如果没有数据可读，则read方法一直阻塞，直到读到新的数据
	            while ((count=socketChannel.read(byteBuffer))>0){
	                byteBuffer.flip();
	                while (byteBuffer.hasRemaining()){
	                    System.out.print((char) byteBuffer.get());
	                }
	                System.out.println();
	                sendToServer("say hi".getBytes());
	                byteBuffer.clear();
	            }
	        }
	    }
	
	    private void sendToServer(byte[] bytes) throws IOException {
	        byteBuffer.clear();
	        byteBuffer.put(bytes);
	        byteBuffer.flip();
	        socketChannel.write(byteBuffer);
	    }
	}