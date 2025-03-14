# 基于Spring AOP, redission实现方法级别的分布式锁通用组件

## 定义自定义注解

```Java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface DistributeLock {

    public String scene();

    public String key() default DistributeLockConstant.NONE_KEY;

    public String keyExpression() default DistributeLockConstant.NONE_KEY;

    public int expireTime() default DistributeLockConstant.DEFAULT_EXPIRE_TIME;

    public int waitTime() default DistributeLockConstant.DEFAULT_WAIT_TIME;
}
```

## Springboot 切面拦截

```Java
@Aspect
@Component
@Order(Integer.MIN_VALUE)
public class DistributeLockAspect {

    private RedissonClient redissonClient;

    public DistributeLockAspect(RedissonClient redissonClient) {
        this.redissonClient = redissonClient;
    }

    private static final Logger LOG = LoggerFactory.getLogger(DistributeLockAspect.class);

    @Around("@annotation(xx.lock.DistributeLock)")
    public Object process(ProceedingJoinPoint pjp) throws Exception {
        Object response = null;
        Method method = ((MethodSignature) pjp.getSignature()).getMethod();
        DistributeLock distributeLock = method.getAnnotation(DistributeLock.class);

        String key = distributeLock.key();
        if (DistributeLockConstant.NONE_KEY.equals(key)) {
            if (DistributeLockConstant.NONE_KEY.equals(distributeLock.keyExpression())) {
                throw new DistributeLockException("no lock key found...");
            }
            SpelExpressionParser parser = new SpelExpressionParser();
            Expression expression = parser.parseExpression(distributeLock.keyExpression());

            EvaluationContext context = new StandardEvaluationContext();
            Object[] args = pjp.getArgs();
            StandardReflectionParameterNameDiscoverer discoverer
                    = new StandardReflectionParameterNameDiscoverer();
            String[] parameterNames = discoverer.getParameterNames(method);

            if (parameterNames != null) {
                for (int i = 0; i < parameterNames.length; i++) {
                    context.setVariable(parameterNames[i], args[i]);
                }
            }

            key = String.valueOf(expression.getValue(context));
        }

        String scene = distributeLock.scene();
        String lockKey = scene + "#" + key;
        int expireTime = distributeLock.expireTime();
        int waitTime = distributeLock.waitTime();
        RLock rLock= redissonClient.getLock(lockKey);
        try {
            boolean lockResult = false;
            if (waitTime == DistributeLockConstant.DEFAULT_WAIT_TIME) {
                if (expireTime == DistributeLockConstant.DEFAULT_EXPIRE_TIME) {
                    LOG.info(String.format("lock for key : %s", lockKey));
                    rLock.lock();
                } else {
                    LOG.info(String.format("lock for key : %s , expire : %s", lockKey, expireTime));
                    rLock.lock(expireTime, TimeUnit.MILLISECONDS);
                }
                lockResult = true;
            } else {
                if (expireTime == DistributeLockConstant.DEFAULT_EXPIRE_TIME) {
                    LOG.info(String.format("try lock for key : %s , wait : %s", lockKey, waitTime));
                    lockResult = rLock.tryLock(waitTime, TimeUnit.MILLISECONDS);
                } else {
                    LOG.info(String.format("try lock for key : %s , expire : %s , wait : %s", lockKey, expireTime, waitTime));
                    lockResult = rLock.tryLock(waitTime, expireTime, TimeUnit.MILLISECONDS);
                }
            }

            if (!lockResult) {
                LOG.warn(String.format("lock failed for key : %s , expire : %s", lockKey, expireTime));
                throw new DistributeLockException("acquire lock failed... key : " + lockKey);
            }


            LOG.info(String.format("lock success for key : %s , expire : %s", lockKey, expireTime));
            response = pjp.proceed();
        } catch (Throwable e) {
            throw new Exception(e);
        } finally {
            if (rLock.isHeldByCurrentThread()) {
                rLock.unlock();
                LOG.info(String.format("unlock for key : %s , expire : %s", lockKey, expireTime));
            }
        }
        return response;
    }
}
```