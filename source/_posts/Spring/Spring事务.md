---
title: Spring事务
date: 2020-12-22 20:04:20
tags:
categories: Spring
---



# Spring 事务

## 事务特性

+ 原子性
+ 一致性
+ 隔离性
+ 持久性

## 事务的隔离级别

隔离级别是指若干个并发的事务之间的隔离程度。TransactionDefinition 接口中定义了五个表示隔离级别的常量：

- TransactionDefinition.ISOLATION_DEFAULT：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。
- TransactionDefinition.ISOLATION_READ_UNCOMMITTED：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读和不可重复读，因此很少使用该隔离级别。
- TransactionDefinition.ISOLATION_READ_COMMITTED：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
- TransactionDefinition.ISOLATION_REPEATABLE_READ：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。
- TransactionDefinition.ISOLATION_SERIALIZABLE：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

## 事务的传播行为

事务传播行为用来描述由某一个事务传播行为修饰的方法被嵌套进另一个方法的时事务如何传播。

```java
ServiceA {   
     void methodA() {
         ServiceB.methodB();
     }
}

ServiceB { 
     void methodB() {
     }      
}
```

- Spring中的7个事务传播行为:

  | 事务行为                  | 说明                                                         |
  | ------------------------- | ------------------------------------------------------------ |
  | PROPAGATION_REQUIRED      | 支持当前事务，假设当前没有事务。就新建一个事务               |
  | PROPAGATION_SUPPORTS      | 支持当前事务，假设当前没有事务，就以非事务方式运行           |
  | PROPAGATION_MANDATORY     | 支持当前事务，假设当前没有事务，就抛出异常                   |
  | PROPAGATION_REQUIRES_NEW  | 新建事务，假设当前存在事务。把当前事务挂起                   |
  | PROPAGATION_NOT_SUPPORTED | 以非事务方式运行操作。假设当前存在事务，就把当前事务挂起     |
  | PROPAGATION_NEVER         | 以非事务方式运行，假设当前存在事务，则抛出异常               |
  | PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |

## 事务超时

事务超时是指一个事务允许执行的最长时间，如果该事件还未完成就自动回滚事务。在TransactionDefinition中以int的值来表示超时的时间。

## 事务回滚规则

正常情况下，如果事务中抛出了未检查异常（继承子RuntimeException的异常），则将事务回滚。如果没有抛出任何异常。或者抛出已检查异常，则仍然提交事务。但是我们也可以根据事务在抛出某些未检查异常时仍然提交事务，或者抛出已检查异常时回滚。

## Spring 事务的配置方式

Spring 支持两种管理方式，编程式和声明式

### 编程式事务管理

编程式事务管理是侵入型事务管理。像Hibernate这个框架中在处理事务的时候需要调用beginTransaction()、commit()、rollback()等事务管理相关的方法，这就是编程式事务管理。

__在Spring中一般采用TransactionTemplate的编程式事务管理__

TransactionTemplate 的 execute() 方法有一个 TransactionCallback 类型的参数，该接口中定义了一个 doInTransaction() 方法，通常我们以匿名内部类的方式实现 TransactionCallback 接口，并在其 doInTransaction() 方法中书写业务逻辑代码。这里可以使用默认的事务提交和回滚规则，这样在业务代码中就不需要显式调用任何事务管理的 API。doInTransaction() 方法有一个TransactionStatus 类型的参数，我们可以在方法的任何位置调用该参数的 setRollbackOnly() 方法将事务标识为回滚的，以执行事务回滚。

根据默认规则，如果在执行回调方法的过程中抛出了未检查异常，或者显式调用了TransacationStatus.setRollbackOnly() 方法，则回滚事务；如果事务执行完成或者抛出了 checked 类型的异常，则提交事务。

TransactionCallback 接口有一个子接口 TransactionCallbackWithoutResult，该接口中定义了一个 doInTransactionWithoutResult() 方法，TransactionCallbackWithoutResult 接口主要用于事务过程中不需要返回值的情况。当然，对于不需要返回值的情况，我们仍然可以使用 TransactionCallback 接口，并在方法中返回任意值即可。

### 声明式事务管理

Spring 的声明式事务管理在底层是建立在 AOP 的基础之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

优点

+ 不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明（或通过等价的基于标注的方式），便可以将事务规则应用到业务逻辑中。

缺点

+ 无法做到细粒度，只能作用到方法级别。无法做到代码块级别。

具体的实现的时候有很多种方式，这里列举两个比较常用的方式

#### 常见的几种声明方式

##### 基于   _TX_  命名空间

Spring 2.x 引入了 TX  命名空间，结合使用 TX  命名空间，带给开发人员配置声明式事务的全新体验，配置变得更加简单和灵活。另外，得益于 TX 命名空间的切点表达式支持，声明式事务也变得更加强大。

下面是一个配置的文件

```java
/**
 * Description:
 *      事务配置tx
 * @version v1.0.0
 * @Author xiayu
 * @Date 2020/10/28 16:01
 */
@Configuration
public class TxAnoConfiguration {

    private static final int TX_METHOD_TIMEOUT = 5;

    private static final String AOP_POINTCUT_EXPRESSION = "execution (* com.xiayu.springboot_demo.service.*.*(..))";

    /*事务拦截类型*/
    @Bean("txSource")
    public TransactionAttributeSource transactionAttributeSource(){
        NameMatchTransactionAttributeSource source = new NameMatchTransactionAttributeSource();
        /*只读事务，不做更新操作*/
        RuleBasedTransactionAttribute readOnlyTx = new RuleBasedTransactionAttribute();
        readOnlyTx.setReadOnly(true);
        readOnlyTx.setPropagationBehavior(TransactionDefinition.PROPAGATION_NOT_SUPPORTED );

        /*当前存在事务就使用当前事务，当前不存在事务就创建一个新的事务*/
        RuleBasedTransactionAttribute requiredTx = new RuleBasedTransactionAttribute(TransactionDefinition.PROPAGATION_REQUIRED,
                Collections.singletonList(new RollbackRuleAttribute(Exception.class)));
        requiredTx.setTimeout(TX_METHOD_TIMEOUT);
        Map<String, TransactionAttribute> txMap = new HashMap<>();

        txMap.put("create*", requiredTx);
        txMap.put("insert*", requiredTx);
        txMap.put("update*", requiredTx);
        txMap.put("delete*", requiredTx);
        txMap.put("get*", readOnlyTx);
        txMap.put("select*", readOnlyTx);
        txMap.put("list*", readOnlyTx);
        source.setNameMap( txMap );

        return source;
    }

    /**切面拦截规则 参数会自动从容器中注入*/
    @Bean
    public AspectJExpressionPointcutAdvisor pointcutAdvisor(TransactionInterceptor txInterceptor){
        AspectJExpressionPointcutAdvisor pointcutAdvisor = new AspectJExpressionPointcutAdvisor();
        pointcutAdvisor.setAdvice(txInterceptor);
        pointcutAdvisor.setExpression(AOP_POINTCUT_EXPRESSION);
        return pointcutAdvisor;
    }
    /*事务拦截器*/
    @Bean("txInterceptor")
    TransactionInterceptor getTransactionInterceptor(PlatformTransactionManager tx){
        return new TransactionInterceptor(tx , transactionAttributeSource()) ;
    }

}

```

##### 基于 @Transactional

Spring 2.x 还引入了基于 Annotation 的方式，具体主要涉及@Transactional 标注。

@Transactional 可以作用于接口、接口方法、类以及类方法上。



基于 tx 命名空间和基于 @Transactional 的事务声明方式各有优缺点。基于 tx 的方式，其优点是与切点表达式结合，功能强大。利用切点表达式，一个配置可以匹配多个方法，而基于 @Transactional 的方式必须在每一个需要使用事务的方法或者类上用 @Transactional 标注，尽管可能大多数事务的规则是一致的，但是对 @Transactional 而言，也无法重用，必须逐个指定。另一方面，基于 @Transactional 的方式使用起来非常简单明了，没有学习成本。开发人员可以根据需要，任选其中一种使用，甚至也可以根据需要混合使用这两种方式。

## 小结

- 基于 TransactionDefinition、PlatformTransactionManager、TransactionStatus 编程式事务管理是 Spring 提供的最原始的方式，通常我们不会这么写，但是了解这种方式对理解 Spring 事务管理的本质有很大作用。
- 基于 TransactionTemplate 的编程式事务管理是对上一种方式的封装，使得编码更简单、清晰。
- 基于 TransactionInterceptor 的声明式事务是 Spring 声明式事务的基础，通常也不建议使用这种方式，但是与前面一样，了解这种方式对理解 Spring 声明式事务有很大作用。
- 基于 TransactionProxyFactoryBean 的声明式事务是上中方式的改进版本，简化的配置文件的书写，这是 Spring 早期推荐的声明式事务管理方式，但是在 Spring 2.0 中已经不推荐了。
- 基于tx 和 aop 命名空间的声明式事务管理是目前推荐的方式，其最大特点是与 Spring AOP 结合紧密，可以充分利用切点表达式的强大支持，使得管理事务更加灵活。
- 基于 @Transactional 的方式将声明式事务管理简化到了极致。开发人员只需在配置文件中加上一行启用相关后处理 Bean 的配置，然后在需要实施事务管理的方法或者类上使用 @Transactional 指定事务规则即可实现事务管理，而且功能也不必其他方式逊色。

