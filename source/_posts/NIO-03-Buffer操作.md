---
title: Buffer操作
date: 2018-02-27 14:03:47
tags: NIO
---

### BIO
* jdk1.0

### NIO
* jdk1.4新增
* java.nio.ByteBuffer
* java.nio.channels.ServerSocketChannel
* java.nio.channels.SocketChannel
* java.nio.channels.Selector
* java.nio.channels.SelectionKey
* java.net.InetSocketAddress

### AIO
* jdk1.7新增
* java.nio.channels.AsynchronousServerSocketChannel
* java.nio.channels.AsynchronousSocketChannel
* java.nio.channels.AsynchronousChannelGroup
* java.nio.channels.CompletionHandler

---

### buffer
* 数据容器

### channel
* 表示到能够执行IO操作的实体的连接

---

#### 容量
* 缓冲区所包含的元素的数量，不能为负且不能更改

#### 限制
* 第一个不应该读取或写入的元素的索引，不能为负且不能大于容量

#### 位置
* 下一个要读取或写入的元素的索引，不能为负，不能大于限制

---

#### 相对操作
* 从当前位置开始，然后将位置增加所传输的元素数

#### 绝对操作
* 采用显式元素索引，该操作不影响位置

---

#### clear
* 使缓冲区为一系列新的通道读取或相对放置 操作做好准备：它将限制设置为容量大小，将位置设置为 0。

#### flip
* 使缓冲区为一系列新的通道写入或相对获取 操作做好准备：它将限制设置为当前位置，然后将位置设置为 0。 

```
    public final Buffer flip() {
        limit = position;
        position = 0;
        mark = -1;
        return this;
    }
```

#### rewind
* 使缓冲区为重新读取已包含的数据做好准备：它使限制保持不变，将位置设置为 0。

---

### 基本操作

```
		// 1 基本操作
		//创建指定长度的缓冲区
		IntBuffer buf = IntBuffer.allocate(10);
		buf.put(13);// position位置：0 - > 1
		buf.put(21);// position位置：1 - > 2
		buf.put(35);// position位置：2 - > 3
		//把位置复位为0，也就是position位置：3 - > 0
		buf.flip(); //limit=position, position=0,准备开始读
		System.out.println("使用flip复位：" + buf);
		System.out.println("容量为: " + buf.capacity());	// 10, 容量一旦初始化后不允许改变（warp方法包裹数组除外）
		System.out.println("限制为: " + buf.limit());		// 3,  由于只装载了三个元素,所以可读取或者操作的元素为3 则limit=3
		
		
		//绝对操作不影响position
		System.out.println("获取下标为1的元素：" + buf.get(1)); //21
		System.out.println("get(index)方法，position位置不改变：" + buf);
		buf.put(1, 4);
		System.out.println("put(index, change)方法，position位置不变：" + buf);;
		
		for (int i = 0; i < buf.limit(); i++) {
			//调用get方法会使其缓冲区位置（position）向后递增一位
			System.out.print(buf.get() + "\t");
		}
		System.out.println("buf对象遍历之后为: " + buf);
```

### warp方法的使用

```
		// 2 wrap方法使用
		
		//  wrap方法会包裹一个数组: 一般这种用法不会先初始化缓存对象的长度，因为没有意义，最后还会被wrap所包裹的数组覆盖掉。 
		//  并且wrap方法修改缓冲区对象的时候，数组本身也会跟着发生变化。                     
		int[] arr = new int[]{1,2,5};
		IntBuffer buf1 = IntBuffer.wrap(arr);
		System.out.println(buf1);
		
		IntBuffer buf2 = IntBuffer.wrap(arr, 0 , 2);
		//这样使用表示容量为数组arr的长度，但是可操作的元素只有实际进入缓存区的元素长度
		System.out.println(buf2);
		
		for(int i=0; i<buf1.limit(); i++){
			System.out.print(buf1.get() + "\t");
		}
		System.out.println(buf1);
		
		/*for(int i=0; i<buf2.limit(); i++){
			System.out.print(buf2.get() + "\t");
		}
		System.out.println(buf2);*/
```

### 其他方法

```
        // 3 其他方法
		IntBuffer buf1 = IntBuffer.allocate(10);
		int[] arr = new int[]{1,2,5};
		buf1.put(arr);
		System.out.println(buf1);
		
		//一种复制方法
		IntBuffer buf3 = buf1.duplicate();
		System.out.println(buf3);
		
		//设置buf1的位置属性
		//buf1.position(1);
		buf1.flip();
		System.out.println(buf1);
		
		System.out.println("可读数据为：" + buf1.remaining());
		
		int[] arr2 = new int[buf1.remaining()];
		
		//将缓冲区数据放入arr2数组中去
		buf1.get(arr2);
		for(int i : arr2){
			System.out.print(Integer.toString(i) + ",");
		}
```
