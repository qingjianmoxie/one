#### Common

* 当数据包（datagram）到达网卡,网卡会产生一个中断，kernal 响应中断，并从网卡缓冲区读出数据放进协议栈中，kernal 在满足一定条件下（一般是用户进程Blocking 中），调用用户进程。数据量大的时候，kernal 让网卡做interrupt coalescing,让网卡做中断合并。数据量再大，kernal 直接禁掉网卡的中断，采用轮询处理来不停的读取网卡缓存区的数据。

- sync IO

- kernal call

- 标准IO 

  ***Block Defination***

  * Waiting for kernal get the data from  the I/O  to be ready
  * copy data from kernal to process

  ***Asynchronous Defination***

  * A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes;
  * An asynchronous I/O operation does not cause the requesting process to be blocked;

#### IO Model

* Blocking IO 

  ```sequence
  Title: Blocking IO
  User -> kernal: (1)system call  recv() 
  Note OVER kernal: (2)waiting \n Datagram 
  Note OVER kernal: (3)Datagram \n arrived
  Note left of User: 进程阻塞\n 从(1)->(5)
  Note OVER kernal: (4)复制数据包到\n用户空间
  kernal -> User:  (5)返回
  ```

  

* Non Blocking IO

  ```sequence
  User -> kernal: (1)recv
  kernal -> User: (2)数据未准备好
  User -> kernal: (3)recv
  Note left of User:进程阻塞在 \n select 操作中 
  kernal -> User: (4)数据未准备好
  User -> kernal: (5)recv
  Note over  kernal: (6)数据准备好
  Note left of User: 进程阻塞\n从(1)->(8)
  Note over kernal: (7)复制数据包到\n用户空间
  kernal -> User:  (8)返回
  ```

  

* IO Multiplexing

   ```sequence
  User -> kernal: (1)select() 
  Note over  kernal : (2)数据\n未准备好
  Note left of User: select \n poll \n epoll \n 进程阻塞
  Note over kernal: (3)数据\n准备好
  kernal -> User: (4)返回可读事件
  User -> kernal : (5)recv
  Note left of User: 进程阻塞 \n 从(5)->(7)
  Note over kernal: (6)复制数据包到\n用户空间
  kernal -> User:  (7)返回
  ```

  

* Singal-driven IO

  ```sequence
  User -> kernal: (1)注册信号
  kernal -> User: (2)返回
  Note left of User: 进程\n继续\n运行
  Note right of kernal : (3)数据\n未准备好
  Note right of kernal: (4)数据\n准备好
  kernal -> User: (5)返回SIGIO
  User -> kernal : (6)recv
  Note left of User: 进程阻塞\n(6)->(8)
  Note right of kernal: (7)复制\n数据包到\n用户空间
  kernal -> User:  (8)返回
  ```

  

* Asynchronous  IO

  ```sequence
  User -> kernal: aio_read()
  kernal -> User: 返回
  Note right of kernal : 数据\n未准备好
  Note right of kernal: 数据\n准备好
  Note left of User: 进程\n继续\n运行
  Note right of kernal: 复制\n数据包到\n用户空间
  kernal -> User:  返回对应aio_read()的信号
  ```

  

#### Multiplexing 

##### ***Select*** 

- Array （interator）
- fd size 1024  （16个long 素组 ，一共1024比特位）   
- Copy  fd set from user space to kernal space when each select And Copy fd set from kernal space to user space when event happen

***poll***

- List interator
- not fd limit
- Copy  fd set from user space to kernal space when each select

***epoll***

- Copy event Data once to kernal event Table

- RBTree

***Trigger***

* 针对与epoll_wait() 读数据，user space buf , kernal buffer space.每次用户读取的数据字节满了就返回，即data 100byte, buf 50byte,

  need get Data 2 Times; level Trigger call 2  times, edge trigger 阻塞式，读取50字节后不再读取，直到下个事件达到，再读取下50个字节(如果对方需要这边对应响应,再继续发送的话,会存在卡死的状态). edge trigger 非阻塞式读取50字节后返回的大于 0说明还有数据,继续读,直读到 0 或者 -1

* Level Trigger 每个事件没处理完下次遍历到还提示处理
* Edge Trigger 事件只提示一次,不处理再下次触发前不在提示.
  * Blocking IO
  * Non Blocking IO 系统对于
  * 这里的 Blocking 和 Non Blocking 是针对与读取缓冲区数据的响应，blocking 如果无数据会一直等待，non blocking 会立即返回 -1

