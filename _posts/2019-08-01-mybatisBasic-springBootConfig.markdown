---
layout: post
title:  "Mybatis 基础及 springBoot 中的配置"
date:   2019-08-01 21:20:56 +0800
categories: Summary
tags: SpringBoot
author: Eric
description: 了解mybatis的事务管理, 及如何在SpringBoot2.X中进行配置
---

# mybatis事务管理器配置

-----  
#### 一、事务管理类型   

1. 事务隔离级别   

    隔离级别是指若干个并发的事务之间的隔离程度。     
    TransactionDefinition 接口中定义了五个表示隔离级别的常量：   

  - TransactionDefinition.ISOLATION_DEFAULT：这是默认值, 表示使用底层数据库的默认隔离级别。对大部分数据库而言, 通常这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。
  - TransactionDefinition.ISOLATION_READ_UNCOMMITTED：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读, 不可重复读和幻读, 因此很少使用该隔离级别。比如PostgreSQL实际上并没有此级别。
  - TransactionDefinition.ISOLATION_READ_COMMITTED：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读, 这也是大多数情况下的推荐值。（锁定正在读取的行）
  - TransactionDefinition.ISOLATION_REPEATABLE_READ：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询, 并且每次返回的记录都相同。该级别可以防止脏读和不可重复读。（锁定所读取的所有行）
  - TransactionDefinition.ISOLATION_SERIALIZABLE：所有的事务依次逐个执行, 这样事务之间就完全不可能产生干扰, 也就是说, 该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。（锁表）   

2. 事务传播行为

    所谓事务的传播行为是指, 如果在开始当前事务之前, 一个事务上下文已经存在, 此时有若干选项可以指定一个事务性方法的执行行为。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：     

  - TransactionDefinition.PROPAGATION_REQUIRED：如果当前存在事务, 则加入该事务；如果当前没有事务, 则创建一个新的事务。这是默认值。
  - TransactionDefinition.PROPAGATION_REQUIRES_NEW：创建一个新的事务, 如果当前存在事务, 则把当前事务挂起。
  - TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务, 则加入该事务；如果当前没有事务, 则以非事务的方式继续运行。
  - TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行, 如果当前存在事务, 则把当前事务挂起。
  - TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行, 如果当前存在事务, 则抛出异常。
  - TransactionDefinition.PROPAGATION_MANDATORY：如果当前存在事务, 则加入该事务；如果当前没有事务, 则抛出异常。
  - TransactionDefinition.PROPAGATION_NESTED：如果当前存在事务, 则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务, 则该取值等价于&emsp;TransactionDefinition.PROPAGATION_REQUIRED。   

3. 事务超时

    所谓事务超时, 就是指一个事务所允许执行的最长时间, 如果超过该时间限制但事务还没有完成, 则自动回滚事务。在 TransactionDefinition 中以 int 的值来表示超时时间, 其单位是秒。
    默认设置为底层事务系统的超时值, 如果底层数据库事务系统没有设置超时值, 那么就是none, 没有超时限制。

4. 事务只读属性

    只读事务用于客户代码只读但不修改数据的情形, 只读事务用于特定情景下的优化, 比如使用Hibernate的时候。
    默认为读写事务。

5. spring事务回滚规则

    指示spring事务管理器回滚一个事务的推荐方法是在当前事务的上下文内抛出异常。spring事务管理器会捕捉任何未处理的异常, 然后依据规则决定是否回滚抛出异常的事务。

6. 默认配置
    默认配置下, spring只有在抛出的异常为运行时unchecked异常时才回滚该事务, 也就是抛出的异常为RuntimeException的子类(Errors也会导致事务回滚), 而抛出checked异常则不会导致事务回滚。
    可以明确的配置在抛出那些异常时回滚事务, 包括checked异常。也可以明确定义那些异常抛出时不回滚事务。
    ackOnly()后你所能执行的唯一操作就是回滚。   
        
------  

#### 二、pom依赖   
1. \<parent>\</parent>标签引入springboot父依赖
2. 使用了spring和mybatis集成包, 整合spring和mybatis
3. mysql数据库驱动包
4. 序列化支持fastjson   


| 属性                 | 类型                             | 描述                                 | 属性                 |
| ---------------------- | ---------------------------------- | -------------------------------------- | ---------------------- |
| value                  | String                             | 可选的限定描述符, 指定使用的事务管理器 | value                  |
| propagation            | enum: Propagation                  | 可选的事务传播行为设置      | propagation            |
| isolation              | enum: Isolation                    | 可选的事务隔离级别设置      | isolation              |
| readOnly               | boolean                            | 读写或只读事务, 默认读写   | readOnly               |
| timeout                | int (in seconds granularity)       | 事务超时时间设置               | timeout                |
| rollbackFor            | Class对象数组, 必须继承自Throwable | 导致事务回滚的异常类数组   | rollbackFor            |
| rollbackForClassName   | 类名数组, 必须继承自Throwable | 导致事务回滚的异常类名字数组 | rollbackForClassName   |
| noRollbackFor          | Class对象数组, 必须继承自Throwable | 不会导致事务回滚的异常类数组 | noRollbackFor          |
| noRollbackForClassName | 类名数组, 必须继承自Throwable | 不会导致事务回滚的异常类名字数组 | noRollbackForClassName |   

------  

#### 三、两个重要注解   
- @Transactional
用于service层注解, 标注在何处使用事务, 及使用的形式。   

{% highlight ruby %} 
@Transactional(readOnly = true)
public class DefaultFooService implements FooService {
 
  public Foo getFoo(String fooName) {
    // do something
  }
 
  // these settings have precedence for this method
  //方法上注解属性会覆盖类注解上的相同属性
  @Transactional(readOnly = false, propagation = Propagation.REQUIRES_NEW)
  public void updateFoo(Foo foo) {
    // do something
  }
}
{% endhighlight %} 


- @EnableTransactionManagement
用于启动事务管理器, 在Application类中开启事务管理    

-----   

#### 四、资料    
> 脏读 ：脏读就是指当一个事务正在访问数据, 并且对数据进行了修改, 而这种修改还没有提交到数据库中, 这时, 另外一个事务也访问这个数据, 然后使用了这个数据。   

>不可重复读 ：是指在一个事务内, 多次读同一数据。在这个事务还没有结束时, 另外一个事务也访问该同一数据。那么, 在第一个事务中的两 次读数据之间, 由于第二个事务的修改, 那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的, 因此称为是不 可重复读。例如, 一个编辑人员两次读取同一文档, 但在两次读取之间, 作者重写了该文档。当编辑人员第二次读取文档时, 文档已更改。原始读取不可重复。如果 只有在作者全部完成编写后编辑人员才可以读取文档, 则可以避免该问题。    

>幻读 : 是指当事务不是独立执行时发生的一种现象, 例如第一个事务对一个表中的数据进行了修改, 这种修改涉及到表中的全部数据行。 同时, 第二个事务也修改这个表中的数据, 这种修改是向表中插入一行新数据。那么, 以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行, 就好象 发生了幻觉一样。例如, 一个编辑人员更改作者提交的文档, 但当生产部门将其更改内容合并到该文档的主复本时, 发现作者已将未编辑的新材料添加到该文档中。 如果在编辑人员和生产部门完成对原始文档的处理之前, 任何人都不能将新材料添加到文档中, 则可以避免该问题      


#### 引用:

###### [spring,mybatis事务管理 隔离级别与事务传播][blog1-url]   
###### [什么是脏读, 不可重复读, 幻读][blog2-url]

[blog1-url]: https://blog.csdn.net/u014001866/article/details/52817643
[blog2-url]: https://www.cnblogs.com/phoebus0501/archive/2011/02/28/1966709.html