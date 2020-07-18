#### @Transactional

- **作用于类**：当把`@Transactional 注解放在类上时，表示所有该类的`public方法都配置相同的事务属性信息。
- **作用于方法**：当类配置了`@Transactional`，方法也配置了`@Transactional`，方法的事务会覆盖类的事务配置信息。
- **作用于接口**：不推荐这种使用方法，因为一旦标注在Interface上并且配置了Spring AOP 使用CGLib动态代理，将会导致`@Transactional`注解失效



##### propagation属性

propagation` 代表事务的传播行为，默认值为 `Propagation.REQUIRED

`Propagation.REQUIRED`：如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。

`Propagation.REQUIRES_NEW`：**重新创建一个新的事务**，如果当前存在事务，暂停当前的事务。**(** 当类A中的 a 方法用默认`Propagation.REQUIRED`模式，类B中的 b方法加上采用 `Propagation.REQUIRES_NEW`模式，然后在 a 方法中调用 b方法操作数据库，然而 a方法抛出异常后，b方法并没有进行回滚，因为`Propagation.REQUIRES_NEW`会暂停 a方法的事务 **)**



##### isolation 属性

`isolation` ：事务的隔离级别，默认值为 `Isolation.DEFAULT`。

**TransactionDefinition.ISOLATION_DEFAULT:** 使用后端数据库**默认的隔离级别**，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.

##### timeout 属性

`timeout` ：事务的**超时时间**，默认值为 -1。如果超过该时间限制但事务还没有完成，则自动回滚事务。

##### readOnly 属性

`readOnly` ：指定事务是否为只读事务，默认值为 false

##### rollbackFor 属性

`rollbackFor` ：用于指定能够触发事务回滚的异常类型，可以指定多个异常类型。

##### **noRollbackFor**属性

`noRollbackFor`：抛出指定的异常类型，不回滚事务，也可以指定多个异常类型。



### @Transactional失效场景

如果`Transactional`注解应用在**非`public` 修饰**的方法上，Transactional将会失效。

**注意：`protected`、`private` 修饰的方法上使用 `@Transactional` 注解，虽然事务无效，但不会有任何报错，这是我们很容犯错的一点。**



Spring默认抛出了未检查`unchecked`异常（继承自 `RuntimeException` 的异常）**或者 `Error`**才回滚事务



若在目标方法中抛出的异常是 `rollbackFor` 指定的异常的子类，事务同样会回滚。



#### 异常被你的 catch“吃了”导致@Transactional失效











