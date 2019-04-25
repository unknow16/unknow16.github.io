---
title: NP-08-Buffer操作
date: 2018-09-11 15:22:33
tags: NetworkProgramming
---


## Buffer
#### Buffer实现
- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer
- MappedByteBuffer

这些Buffer覆盖了你能通过IO发送的基本数据类型：byte, short, int, long, float, double 和 char。MappedByteBuffer，用于表示内存映射文件。

#### 三个属性
- capacity: 作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。
- position: 当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1。当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0. 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。
- limit： 在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position）

#### 向Buffer中写数据有两种方式

1. 从Channel写到Buffer。
1. 通过Buffer的put()方法写到Buffer里。

#### 从Buffer中读取数据有两种方式

1. 从Buffer读取数据到Channel。
1. 使用get()方法从Buffer中读取数据。

#### flip()方法
flip方法将Buffer从写模式切换到读模式。调用flip()方法会将position设回0，并将limit设置成之前position的值。

换句话说，position现在用于标记读的位置，limit表示之前写进了多少个byte、char等 —— 现在能读取多少个byte、char等。

#### rewind()方法
Buffer.rewind()将position设回0，所以你可以重读Buffer中的所有数据。limit保持不变，仍然表示能从Buffer中读取多少个元素（byte、char等）。

#### clear()与compact()方法
一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。

如果调用的是clear()方法，position将被设回0，limit被设置成 capacity的值。换句话说，Buffer 被清空了。Buffer中的数据并未清除，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。

如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。

如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。

compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。

#### mark()与reset()方法
通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。例如：


```
buffer.mark();

//call buffer.get() a couple of times, e.g. during parsing.

buffer.reset();  //set position back to mark.
```

## 基本操作

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

## warp方法的使用

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

## 其他方法

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
