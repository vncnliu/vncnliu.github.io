---
title: junit多线程测试
date: 2016-04-18 14:50:21
category:
- 程序猿的attack
tag:
- junit
- 多线程
---

工作中难免要考虑一些方法是不是线程安全的，这时可以用junit进行简单的多线程测试。方式有很多种，可以使用grobo-utils不过grobo-utils没有放到maven公共仓库里，所以在maven项目中使用时还要把jar包打到本地仓库里，难免有些麻烦。
其实还可以通过CountDownLatch很方便的实现。比如我写一个基于LinkedList简单的队列
```java
package common;

import com.google.common.base.Strings;

import java.util.LinkedList;
import java.util.List;

/**
 * All right reserved.
 * Created by vncnliu@gmail.com on 2016/4/12.
 */
public class LightQueueSynced {

    private static List<String> QUEUE_LIST = new LinkedList<String>();

    public static List<String> getQueueList(){
        return QUEUE_LIST;
    }

    public static synchronized void queue(String key){
        //System.out.println("add:"+seatNo);
        if(!Strings.isNullOrEmpty(key)&&!QUEUE_LIST.contains(key)){
            QUEUE_LIST.add(key);
        }
    }

    public static synchronized void remove(String key){
        QUEUE_LIST.remove(key);
    }

    public static synchronized String pollFirst(){
        if(QUEUE_LIST.size()==0){
            return null;
        }
        String seatNo = QUEUE_LIST.get(0);
        if(!Strings.isNullOrEmpty(seatNo)){
            QUEUE_LIST.remove(0);
        }
        return seatNo;
    }
}
```
我要测试一下队列是不是线程安全的
创建一个junit测试类
```java
package common;

import org.junit.Test;

import java.util.concurrent.CountDownLatch;

/**
 * All right reserved.
 * Created by vncnliu@gmail.com on 2016/4/15.
 */
public class TestLightQueueSynced {

    @Test
    public void test() throws InterruptedException {
        CountDownLatch main = new CountDownLatch(1);//控制子线程等待主线程
        CountDownLatch part = new CountDownLatch(20);//控制主线程等待子线程全部执行完后再继续
        for (int i = 0; i < 10; i++) {
            ThreadProducer producer = new ThreadProducer(main,part);
            producer.start();
        }
        for (int i = 0; i < 10; i++) {
            ThreadConsumer consumer = new ThreadConsumer(main,part);
            consumer.start();
        }
        main.countDown();
        part.await();
        System.out.println(LightQueueSynced.getQueueList().size());
        System.out.println(LightQueueSynced.getQueueList());
    }


    class ThreadProducer extends Thread {

        CountDownLatch main = null;
        CountDownLatch part = null;
        public ThreadProducer(CountDownLatch main, CountDownLatch part) {
            this.main = main;
            this.part = part;
        }

        @Override
        public void run() {
            try {
                this.main.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            LightQueueSynced.queue("test"+Thread.currentThread().getId());
            part.countDown();
        }
    }

    class ThreadConsumer extends Thread {

        CountDownLatch main = null;
        CountDownLatch part = null;
        public ThreadConsumer(CountDownLatch main, CountDownLatch part) {
            this.main = main;
            this.part = part;
        }

        @Override
        public void run() {
            try {
                this.main.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(LightQueueSynced.pollFirst());
            part.countDown();
        }
    }
}
```
CountDownLatch.await()的说明如下
```java
    /**
     * Causes the current thread to wait until the latch has counted down to
     * zero, unless the thread is {@linkplain Thread#interrupt interrupted}.
     *
     * <p>If the current count is zero then this method returns immediately.
     *
     * <p>If the current count is greater than zero then the current
     * thread becomes disabled for thread scheduling purposes and lies
     * dormant until one of two things happen:
     * <ul>
     * <li>The count reaches zero due to invocations of the
     * {@link #countDown} method; or
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread.
     * </ul>
     *
     * <p>If the current thread:
     * <ul>
     * <li>has its interrupted status set on entry to this method; or
     * <li>is {@linkplain Thread#interrupt interrupted} while waiting,
     * </ul>
     * then {@link InterruptedException} is thrown and the current thread's
     * interrupted status is cleared.
     *
     * @throws InterruptedException if the current thread is interrupted
     *         while waiting
     */
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
```
执行结果：
```doc
test19
null
null
null
null
null
null
test18
test15
test21
6
[test13, test17, test16, test20, test14, test12]

Process finished with exit code 0
```
可以看出在保证插入数据不重复的情况下poll出的和剩余的和都是10
如果让主线程sleep(10000)可在JProfiler看到
{% asset_img 1.png JProfiler Threads %}