# 数据库多版本并发控制

## 1. 2PL

简单的两段锁，效率低，容易引起死锁

## 2. T/O (Timestamp Ordering)

为每一次记录记录一个WTS(最近更新该数据的事务的timestamp)和一个RTS（读取过该数据中最大的事务timestamp）
当前事务的时间TTS，若当前事务为读：
若TS < WTS(X) , 则当前事务回滚，其他情况都可以执行读
当前事务为write：
若TS < WTS(X) or TS < RTS(X), 事务回滚，执行失败，其他情况可以执行
不能处理脏读。
读写冲突：读取事务失败回滚
写读冲突：写入事务失败回滚
写写冲突：后写入的事务失败回滚

## 3. OCC(Optimistic Concurrency Control)

三阶段：
	a. write && read  Phase 所有读取的数据会拷贝到本地空间，所有写记录也只会记录到本地空间
	b. validate Phase 事务执行commit的时候，DBMS会先检查事务事务是不是和其他事务有冲突，验证阶段需要对事务修改的数据加锁
	c. write Phase 把本地空间记录的数据apply到DBMS，让其他事务可见，最后释放验证阶段加的锁
验证阶段有两种方法：
    1. Backward Validation
    2. Forward Validation
Backward 会检查当前事务读写的数据集跟已提交的事务是否冲突。
OCC 需要对每条记录额外存储Write Timestamp。
读写冲突： 写入的数据没有commit之前，数据都记录在private workspace， 所以读取的数据是写入前的数据。
写读冲突： 读的事务在validation阶段会发现自己读的数据被最近的commit事务修改过，读事务会回滚。
脏读： 未commit的数据不会被其他事务读到，因此不会发生脏读
写写覆盖： 事务的顺序是按commit的顺序决定的，先commit的事务先完成，后commit的事务会失败。（先validation的事务先commit）
读和写都会将数据copy到private，在validate阶段会验证与时间点之后的事务是否有冲突，如果没有冲突，则write阶段直接写入。不可重复读

## 4. MVCC

多版本，写操作时创建一个版本，读操作时返回最近的版本。

## 5. 2PC in TiKV

第一阶段： prewrite阶段
第二阶段： commit阶段
客户端发请求之后向PD申请事务开始时间戳start_ts， 向leader发送prewrite请求，leader给各个节点发送请求，如果leader收到了所有节点的回复，则执行prewrite。
prewrite阶段完成后，再次向PD申请一个唯一的时间戳作为本次事务的commit_ts， 完成commit阶段。
