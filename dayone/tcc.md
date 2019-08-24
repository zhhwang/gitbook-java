## 用一个例子讲解TCC分支1.x逻辑
### Trying
#### 订单服务
    简单介绍：一个下单流程中，商品可以用红包支付部分金额，资金支付余下金额
    订单支付流程代码
````$xslt
    /**
     * 订单支付流程
     * @param order 订单信息：商品信息、订单号、买家ID、卖家ID...
     * @param redPacketPayAmount 红包支付金额
     * @param capitalPayAmount 资金支付金额
     */
    @Compensable(confirmMethod = "confirmMakePayment", cancelMethod = "cancelMakePayment")
    @Transactional
    public void makePayment(Order order, BigDecimal redPacketPayAmount, BigDecimal capitalPayAmount) {
        // 将订单状态修改为支付中
        order.pay(redPacketPayAmount, capitalPayAmount);
        orderRepository.updateOrder(order);

        // 将资金支付的金额由买家转移到卖家
        capitalTradeOrderService.record(transactionContext: null, buildCapitalTradeOrderDto(order));
        // 将红包支付的金额由买家转移到卖家
        redPacketTradeOrderService.record(transactionContext:null, buildRedPacketTradeOrderDto(order));
    }
````
    其中资金支付服务和红包支付服务的record方法的第一个参数类型为TransactionContext
    在makePayment方法执行之前，将有两层AOP增强
    
##### 外层AOP(第一层)
    @Pointcut("@annotation(xxx.Compensable)")：拦截Compensable注解
    优先级：Ordered.HIGHEST_PRECEDENCE（-2147483648）
````$xslt
    private Object rootMethodProceed(ProceedingJoinPoint pjp) throws Throwable {
        // 开启订单支付事务：创建事务，持久化，并将事务放入ThreadLocal
        // 此时事务状态为TRYING
        transactionConfigurator.getTransactionManager().begin();

        Object returnValue = null;
        try {
            returnValue = pjp.proceed();
        } catch (Throwable tryingException) {
            if (tryingException instanceof OptimisticLockException
                    || ExceptionUtils.getRootCause(tryingException) instanceof OptimisticLockException) {
                // 乐观锁异常导致的失败，将进入重试，超过重试次数将回滚，重试将由recovery任务完成
            } else {
                // 其他异常导致的失败，将会直接回滚
                Transaction transaction = transactionConfigurator.getTransactionManager().getCurrentTransaction();
                logger.warn(String.format("compensable transaction trying failed. transaction content:%s", JSON.toJSONString(transaction)), tryingException);
                // 将订单支付事务设置为取消状态，执行cancel方法，删除事务记录
                transactionConfigurator.getTransactionManager().rollback();
            }

            throw tryingException;
        }

        // 无异常，将订单支付事务设置为完成状态，执行confirm方法，删除事务记录
        transactionConfigurator.getTransactionManager().commit();
        return returnValue;
    }
````
    订单支付事务的cancel方法和confirm方法是在哪里来的呢，这将会在内层AOP中看到
##### 内层AOP(第二层)
    @Pointcut("execution(public * *(xxx.TransactionContext,..))||@annotation(xxx.Compensable)")：拦截Compensable注解
    或者所有第一个参数类型为TransactionContext的public方法
````$xslt
public Object interceptTransactionContextMethod(ProceedingJoinPoint pjp) throws Throwable {
        // ThreadLocal中获取订单支付事务
        Transaction transaction = transactionConfigurator.getTransactionManager().getCurrentTransaction();
        // 如果事务是在TRYING阶段
        if (transaction != null && transaction.getStatus().equals(TransactionStatus.TRYING)) {
            // 在Compensable注解中获取confirm方法名称和cancel方法名称
            Compensable compensable = getCompensable(pjp);
            String confirmMethodName = compensable.confirmMethod();
            String cancelMethodName = compensable.cancelMethod();
            
            // 此处省略部分代码，重在逻辑
            // 创建confirm方法反射调用信息
            // 创建cancel方法反射调用信息
            // 将confirm方法和cancel方法的反射调用信息保存在订单支付事务中(hreadLocal)，并持久化           
        }

        return pjp.proceed(pjp.getArgs());
    }
````    
    此时的confirm方法和cancel方法主要是本地订单状态更改逻辑，分别为：
````$xslt
    public void confirmMakePayment(Order order, BigDecimal redPacketPayAmount, BigDecimal capitalPayAmount) {
        // 将订单状态修改为已完成
        order.confirm();
        orderRepository.updateOrder(order);
    }

    public void cancelMakePayment(Order order, BigDecimal redPacketPayAmount, BigDecimal capitalPayAmount) {
        // 将订单状态修改为取消支付
        order.cancelPayment();
        orderRepository.updateOrder(order);
    }
````
    接下来将执行makePayment方法, 执行到
````$xslt
capitalTradeOrderService.record(transactionContext: null, buildCapitalTradeOrderDto(order));
````
    的时候，将要进行远程调用资金支付服务，然而它的第一个参数为TransactionContext类型，将会被内层AOP增强，
    但，此时内层AOP有了新的逻辑
````$xslt
public Object interceptTransactionContextMethod(ProceedingJoinPoint pjp) throws Throwable {
        // ThreadLocal中获取订单支付事务
        Transaction transaction = transactionConfigurator.getTransactionManager().getCurrentTransaction();
        // 如果事务是在TRYING阶段
        if (transaction != null && transaction.getStatus().equals(TransactionStatus.TRYING)) {
            // 获取类型为TransactionContext的参数的位置
            int position = CompensableMethodUtils.getTransactionContextParamPosition(((MethodSignature) pjp.getSignature()).getParameterTypes());
            // 将参数值设置为新的TransactionContext，此TransactionContext状态为TRYING
            pjp.getArgs()[position] = new TransactionContext(xid, transaction.getStatus().getId());
    
            Object[] tryArgs = pjp.getArgs();
            Class targetClass = ReflectionUtils.getDeclaringType(pjp.getTarget().getClass(), method.getName(), method.getParameterTypes());
            
            // 创建一个confirm的反射调用信息，注意被封装的是远程方法record，第一个参数TransactionContext的状态为CONFIRMING
            Object[] confirmArgs = new Object[tryArgs.length];
            System.arraycopy(tryArgs, 0, confirmArgs, 0, tryArgs.length);
            confirmArgs[position] = new TransactionContext(xid, TransactionStatus.CONFIRMING.getId());
            InvocationContext confirmInvocation = new InvocationContext(targetClass, method.getName(), method.getParameterTypes(), confirmArgs);
            
            // 创意一个cancel的反射调用信息，注意methodName为远程方法record，第一个参数TransactionContext的状态为CANCELLING
            Object[] cancelArgs = new Object[tryArgs.length];
            System.arraycopy(tryArgs, 0, cancelArgs, 0, tryArgs.length);
            cancelArgs[position] = new TransactionContext(xid, TransactionStatus.CANCELLING.getId());
            InvocationContext cancelInvocation = new InvocationContext(targetClass, method.getName(), method.getParameterTypes(), cancelArgs);
    
           
            transactionRepository.update(transaction);
            // 此处省略部分代码，重在逻辑
            // 将反射调用confirm方法和cancel方法的信息添加进订单支付事务中(hreadLocal)
            // 此时订单支付事务中有两个confirm方法和两个cancel方法          
        }

        // 调用资金支付服务的远程方法record，
        return pjp.proceed(pjp.getArgs());
    }
````    
    接下来以资金账户服务为例，来说明子事务的逻辑
#### 资金账户prodiver

````$xslt
    @Override
    @Compensable(confirmMethod = "confirmRecord", cancelMethod = "cancelRecord")
    @Transactional
    public String record(TransactionContext transactionContext, CapitalTradeOrderDto tradeOrderDto) {
        // 保存资金账号订单交易信息
        TradeOrder tradeOrder = new TradeOrder(
                tradeOrderDto.getSelfUserId(),
                tradeOrderDto.getOppositeUserId(),
                tradeOrderDto.getMerchantOrderNo(),
                tradeOrderDto.getAmount()
        );
        tradeOrderRepository.insert(tradeOrder);

        // 检查买家资金账号余额，余额不足将抛出异常，余额充足将用乐观锁更新余额，更新余额失败将抛出异常
        CapitalAccount transferFromAccount = capitalAccountRepository.findByUserId(tradeOrderDto.getSelfUserId());
        transferFromAccount.transferFrom(tradeOrderDto.getAmount());
        capitalAccountRepository.save(transferFromAccount);
        return "success";
    }
````
   record方法既有Compensable注解，第一个参数又为TransactionContext类型，将进入两层AOP增强
##### 外层AOP
````$xslt
 private Object providerMethodProceed(ProceedingJoinPoint pjp, TransactionContext transactionContext) throws Throwable {
        
        switch (TransactionStatus.valueOf(transactionContext.getStatus())) {
            case TRYING:
                // 如果transactionContext状态为TRYING，创建资金账户子事务并持久化，状态为TRYING
                transactionConfigurator.getTransactionManager().propagationNewBegin(transactionContext);
                return pjp.proceed();
            case CONFIRMING:
                // 如果transactionContext状态为CONFIRMING，
                try {
                    // 检测子事务是存在，更新子事务状态为CONFIRMING
                    transactionConfigurator.getTransactionManager().propagationExistBegin(transactionContext);
                    // 在commit方法中执行Compensable中的confirmMethod，删除事务记录
                    transactionConfigurator.getTransactionManager().commit();
                } catch (NoExistedTransactionException excepton) {
                    //the transaction has been commit,ignore it.
                }
                break;
            case CANCELLING:
                // 如果transactionContext状态为CANCELLING，
                try {
                    // 检测子事务是存在，更新子事务状态为CANCELLING
                    transactionConfigurator.getTransactionManager().propagationExistBegin(transactionContext);
                    // 在rollback方法中执行Compensable中的cancelMethod，删除事务记录
                    transactionConfigurator.getTransactionManager().rollback();
                } catch (NoExistedTransactionException exception) {
                    //the transaction has been rollback,ignore it.
                }
                break;
        }

        Method method = ((MethodSignature) (pjp.getSignature())).getMethod();

        return ReflectionUtils.getNullValue(method.getReturnType());
    }
````
##### 内层AOP

````$xslt
public Object interceptTransactionContextMethod(ProceedingJoinPoint pjp) throws Throwable {
        // ThreadLocal中获取订单支付事务
        Transaction transaction = transactionConfigurator.getTransactionManager().getCurrentTransaction();
        // 如果事务是在TRYING阶段
        if (transaction != null && transaction.getStatus().equals(TransactionStatus.TRYING)) {
            // 在Compensable注解中获取confirm方法名称和cancel方法名称
            Compensable compensable = getCompensable(pjp);
            String confirmMethodName = compensable.confirmMethod();
            String cancelMethodName = compensable.cancelMethod();
            
            // 此处省略部分代码，重在逻辑
            // 创建confirm方法反射调用信息
            // 创建cancel方法反射调用信息
            // 将confirm方法和cancel方法的反射调用信息保存在订单支付事务中(hreadLocal)，并持久化           
        }

        return pjp.proceed(pjp.getArgs());
    }
````    
    此时的confirm方法和cancel方法主要是本地订单状态更改逻辑，分别为：
````$xslt
@Transactional
    public void confirmRecord(TransactionContext transactionContext, CapitalTradeOrderDto tradeOrderDto) {
        // 更新交易订单状态
        TradeOrder tradeOrder = tradeOrderRepository.findByMerchantOrderNo(tradeOrderDto.getMerchantOrderNo()); 
        tradeOrder.confirm();
        tradeOrderRepository.update(tradeOrder);
        
        // 卖家资金账号余额增加
        CapitalAccount transferToAccount = capitalAccountRepository.findByUserId(tradeOrderDto.getOppositeUserId());
        transferToAccount.transferTo(tradeOrderDto.getAmount());
        capitalAccountRepository.save(transferToAccount);
    }

    @Transactional
    public void cancelRecord(TransactionContext transactionContext, CapitalTradeOrderDto tradeOrderDto) {
        // 更新交易订单状态
        TradeOrder tradeOrder = tradeOrderRepository.findByMerchantOrderNo(tradeOrderDto.getMerchantOrderNo());
        if(null != tradeOrder && "DRAFT".equals(tradeOrder.getStatus())) {
            tradeOrder.cancel();
            tradeOrderRepository.update(tradeOrder);

            // 买家资金账号余额恢复
            CapitalAccount capitalAccount = capitalAccountRepository.findByUserId(tradeOrderDto.getSelfUserId());
            capitalAccount.cancelTransfer(tradeOrderDto.getAmount());
            capitalAccountRepository.save(capitalAccount);
        }
    }
````
    两层AOP增强之后，开始执行资金账户服务的record方法.
    红包账户服务逻辑相同.
### Confirm

    如果在makePayment方法执行阶段不存在异常，将在订单支付流程的外层AOP中执行事务commit操作
    此例子中，订单支付流程中记录了三个confirm反射调用信息
* confirm1: @Compenable注解中的confirmMakePayment标记的方法，修改订单状态为完成
* confirm2: 资金账号服务远程调用方法record，第一个参数TransactionContext的状态为CONFIRMING
* confirm3: 红包账号服务远程调用方法record，第一个参数TransactionContext的状态为CONFIRMING
       
### Cancel
    如果在makePayment方法执行阶段抛出了异常，将在订单支付流程的外层AOP中执行事务rollback操作
* 在资金账号服务调用前抛出异常，只需回滚@Compenable注解中的cancelMakePayment方法，订单状态改为取消
* 在资金账号服务调用后， 红包账号服务调用前抛出异常
    * @Compenable注解中的cancelMakePayment方法，修改订单状态为取消
    * 资金账号服务远程调用方法record，第一个参数TransactionContext的状态为CANCELLING
* confirm1: 方法，修改订单状态为完成
* confirm2: 资金账号服务远程调用方法record，第一个参数TransactionContext的状态为CONFIRMING
* confirm3: 红包账号服务远程调用方法record，第一个参数TransactionContext的状态为CONFIRMING   

 Confirm 或 Cancel时异常，将不断重试，直成功为止。
    
## TCC 1.2
