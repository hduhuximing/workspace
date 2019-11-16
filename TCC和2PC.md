# TCC和2PC

## 1.什么是2PC

 2PC即两阶段提交协议，是将整个事务流程分为两个阶段，准备阶段（Prepare phase）、提交阶段（commit phase）**，2是指两个阶段，P是指准备阶段，C是指提交阶段。** 

- 准备阶段（Prepare phase）：事务管理器给每个参与者发送Prepare消息，每个数据库参与者在本地执行事务，并写**本地的Undo/Redo日志**，此时事务没有提交。
  （**Undo日志是记录修改前的数据**，用于数据库回滚，**Redo日志是记录修改后的数据**，用于提交事务后写入数据文件）
- 提交阶段（commit phase）：如果事务管理器收到了参与者的执行失败或者超时消息时，直接给每个参与者发送回滚(Rollback)消息；否则，发送提交(Commit)消息；参与者根据事务管理器的指令执行提交或者回滚操作，并释放事务处理过程中使用的锁资源。注意:必须在最后阶段释放锁资源。

 成功情况：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190818231239352.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDA2MjMzOQ==,size_16,color_FFFFFF,t_70)
失败情况：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190818231301474.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDA2MjMzOQ==,size_16,color_FFFFFF,t_70) 

## 2.什么是TCC

TCC是Try、Confirm、Cancel三个词语的缩写，TCC要求每个分支事务实现三个操作：**预处理Try、确认Confirm、撤销Cancel**。**Try操作做业务检查及资源预留**，**Confirm做业务确认操作**，**Cancel实现一个与Try相反的操作即回滚操作**。TM（事务管理器）首先发起所有的分支事务的try操作，任何一个分支事务的try操作执行失败，TM将会发起所有分支事务的Cancel操作，若try操作全部成功，TM将会发起所有分支事务的Confirm操作，其中Confirm/Cancel操作若执行失败，TM会进行重试。
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190818231349577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDA2MjMzOQ==,size_16,color_FFFFFF,t_70)
分支事务失败的情况：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190818231411275.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDA2MjMzOQ==,size_16,color_FFFFFF,t_70) 

TCC分为三个阶段：

- Try 阶段是做业务检查(一致性)及**资源预留(隔离**)，此阶段仅是一个初步操作，它和后续的Confirm 一起才能真正构成一个完整的业务逻辑。

- Confirm 阶段是做确认提交，Try阶段所有分支事务执行成功后开始执行 Confirm。通常情况下，采用TCC则认为 Confirm阶段是不会出错的。即：只要Try成功，Confirm一定成功。**若Confirm阶段真的出错了，需引入重试机制或人工处理。**

- Cancel 阶段是在业务执行错误需要回滚的状态下执行分支事务的业务取消，预留资源释放。通常情况下，采用TCC则认为Cancel阶段也是一定成功的。**若Cancel阶段真的出错了，需引入重试机制或人工处理**。

- TM事务管理器可以实现为独立的服务，也可以让全局事务发起方充当TM的角色，TM独立出来是为了成为公用组件，是为了考虑系统结构和软件复用。

  

  TM在发起全局事务时生成全局事务记录，全局事务ID贯穿整个分布式事务调用链条，用来记录事务上下文，追踪和记录状态，由于Confirm 和cancel失败需进行重试，因此需要实现为幂等，幂等性是指同一个操作无论请求多少次，其结果都相同。
  