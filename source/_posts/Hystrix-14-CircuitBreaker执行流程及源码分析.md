---
title: Hystrix-14-CircuitBreaker执行流程及源码分析
date: 2018-06-13 16:54:29
tags: Hystrix
---

### Hystrix中的断路器是如何决策熔断和记实信息的呢？

官方提供的执行流程图

![image](https://note.youdao.com/yws/api/personal/file/E3BCB3471410473D91BC678875AC1B7E?method=download&shareKey=0416df563391455b89cc80ade631c93b)

每个熔断器默认维护10个bucket,每秒一个bucket,每个blucket记录成功,失败,超时,拒绝的状态，

默认错误超过50%且10秒内超过20个请求进行中断拦截. 

* 断路器接口：HystrixCircuitBreaker,其中内部类对断路器接口进行了实现
```
public interface HystrixCircuitBreaker {
    
    // 每个Hystrix命令的请求都通过该方法判读是否被执行
    public boolean allowRequest();
    // 返回当前断路器是否打开
    public boolean isOpen();
    // 用来闭合断路器
    void markSuccess();
    
    // 每个HystrixCommand有各自的CircuitBreaker
    // 用Map集合维护一个HystrixCommand和它相应的CircuitBreaker的关系
    public static class Factory {}
    
    // 默认断路器实现
    static class HystrixCircuitBreakerImpl implements HystrixCircuitBreaker {}
    
    // 啥也不做的断路器实现
    static class NoOpCircuitBreaker implements HystrixCircuitBreaker {}
}
```

* HystrixCircuitBreaker接口中的静态内部类Factory实现如下：

```
public static class Factory {

        // 用来维护一个HystrixCommand和它相应的CircuitBreaker的关系
        // 该Map的key为HystrixCommand的CommandKey的String值
        private static ConcurrentHashMap<String, HystrixCircuitBreaker> circuitBreakersByCommand = new ConcurrentHashMap<String, HystrixCircuitBreaker>();

        // 获取给定的一个HystrixCommand的相对应的HystrixCircuitBreaker
        // 这是线程安全的，确保每个HystrixCommandKey只有一个HystrixCircuitBreaker
        public static HystrixCircuitBreaker getInstance(HystrixCommandKey key, HystrixCommandGroupKey group, HystrixCommandProperties properties, HystrixCommandMetrics metrics) {
            // this should find it for all but the first time
            HystrixCircuitBreaker previouslyCached = circuitBreakersByCommand.get(key.name());
            if (previouslyCached != null) {
                return previouslyCached;
            }

            // if we get here this is the first time so we need to initialize
            
            // putIfAbsent()方法：如果不存在该键值对，则插入,返回值为null则表示插入成功
            // 返回值不为null,则表示多个线程发生竞态条件，返回其他线程插入的键对应的值
            HystrixCircuitBreaker cbForCommand = circuitBreakersByCommand.putIfAbsent(key.name(), new HystrixCircuitBreakerImpl(key, group, properties, metrics));
            if (cbForCommand == null) {
                // this means the putIfAbsent step just created a new one so let's retrieve and return it
                return circuitBreakersByCommand.get(key.name());
            } else {
                // this means a race occurred and while attempting to 'put' another one got there before
                // and we instead retrieved it and will now return it
                return cbForCommand;
            }
        }

        public static HystrixCircuitBreaker getInstance(HystrixCommandKey key) {
            return circuitBreakersByCommand.get(key.name());
        }

        /**
         * Clears all circuit breakers. If new requests come in instances will be recreated.
         * 
         * 清空map，当新请求来时，重新创建断路器实例。
         */
        /* package */static void reset() {
            circuitBreakersByCommand.clear();
        }
    }
```

* HystrixCircuitBreaker接口中的静态内部类HystrixCircuitBreakerImpl实现，即断路器的默认释实现如下：

```
/* package */static class HystrixCircuitBreakerImpl implements HystrixCircuitBreaker {
        private final HystrixCommandProperties properties;
        private final HystrixCommandMetrics metrics;

        /* track whether this circuit is open/closed at any given point in time (default to false==closed) */
        private AtomicBoolean circuitOpen = new AtomicBoolean(false);

        /* when the circuit was marked open or was last allowed to try a 'singleTest' */
        private AtomicLong circuitOpenedOrLastTestedTime = new AtomicLong();

        protected HystrixCircuitBreakerImpl(HystrixCommandKey key, HystrixCommandGroupKey commandGroup, HystrixCommandProperties properties, HystrixCommandMetrics metrics) {
            this.properties = properties;
            this.metrics = metrics;
        }
        
        // 当circuitOpen为ture，即断路器打开时
        // 调用此方法可关闭断路器，即circuitOpen设置为false，并重置统计数据信息
        public void markSuccess() {
            if (circuitOpen.get()) {
                if (circuitOpen.compareAndSet(true, false)) {
                    // 只影响当前消费者
                    //win the thread race to reset metrics
                    //Unsubscribe from the current stream to reset the health counts stream.  This only affects the health counts view,
                    //and all other metric consumers are unaffected by the reset
                    metrics.resetStream();
                }
            }
        }

        // 每次请求都会进行判断，是否允许通过
        @Override
        public boolean allowRequest() {
            if (properties.circuitBreakerForceOpen().get()) {
                // properties have asked us to force the circuit open so we will allow NO requests
                return false;
            }
            if (properties.circuitBreakerForceClosed().get()) {
                // we still want to allow isOpen() to perform it's calculations so we simulate normal behavior
                isOpen();
                // properties have asked us to ignore errors so we will ignore the results of isOpen and just allow all traffic through
                return true;
            }
            // isOpen()为false,即断路器关闭时，此处返回ture，不执行allowSingleTest方法，允许请求通过
            // isOpen()为true，即断路器打开时，执行allowSingleTest方法
            return !isOpen() || allowSingleTest();
        }

        // 短路器打开时，休眠时间窗时间（默认5秒）到后，返回ture
        // 进而随后allowRequest()返回ture,允许此次请求，否则拒绝请求
        public boolean allowSingleTest() {
            long timeCircuitOpenedOrWasLastTested = circuitOpenedOrLastTestedTime.get();
            // 1) if the circuit is open
            // 2) and it's been longer than 'sleepWindow' since we opened the circuit
            if (circuitOpen.get() && System.currentTimeMillis() > timeCircuitOpenedOrWasLastTested + properties.circuitBreakerSleepWindowInMilliseconds().get()) {
                // We push the 'circuitOpenedTime' ahead by 'sleepWindow' since we have allowed one request to try.
                // If it succeeds the circuit will be closed, otherwise another singleTest will be allowed at the end of the 'sleepWindow'.
                if (circuitOpenedOrLastTestedTime.compareAndSet(timeCircuitOpenedOrWasLastTested, System.currentTimeMillis())) {
                    // if this returns true that means we set the time so we'll return true to allow the singleTest
                    // if it returned false it means another thread raced us and allowed the singleTest before we did
                    return true;
                }
            }
            return false;
        }
        
        // 判断断路器是否打开
        // circuitOpen为ture，直接返回，否则用统计数据计算
        @Override
        public boolean isOpen() {
            if (circuitOpen.get()) {
                // if we're open we immediately return true and don't bother attempting to 'close' ourself as that is left to allowSingleTest and a subsequent successful test to close
                return true;
            }

            // we're closed, so let's see if errors have made us so we should trip the circuit open
            HealthCounts health = metrics.getHealthCounts();

            // 请求总数不大于20次,不打开
            if (health.getTotalRequests() < properties.circuitBreakerRequestVolumeThreshold().get()) {
                // we are not past the minimum volume threshold for the statisticalWindow so we'll return false immediately and not calculate anything
                return false;
            }

            // 错误百分比不大于50%,不打开
            if (health.getErrorPercentage() < properties.circuitBreakerErrorThresholdPercentage().get()) {
                return false;
            } else {
                // our failure rate is too high, trip the circuit
                
                // 10秒内，请求次数达到20次，并错误百分比到50%,设置circuitOpen为ture
                if (circuitOpen.compareAndSet(false, true)) {
                    
                    // 设置断路器打开的时间，为后期休眠时间窗到后关闭断路器作准备
                    circuitOpenedOrLastTestedTime.set(System.currentTimeMillis());
                    return true;
                } else {
                    // How could previousValue be true? If another thread was going through this code at the same time a race-condition could have
                    // caused another thread to set it to true already even though we were in the process of doing the same
                    // In this case, we know the circuit is open, so let the other thread set the currentTime and report back that the circuit is open
                    return true;
                }
            }
        }

    }
```
