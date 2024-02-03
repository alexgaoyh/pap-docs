## 背景

&ensp;&ensp;《一种Java语言下的简单重试实现》近期有一个旧项目的维护，经过排查问题定位在外部接口的请求超时，为了减少改造成本，编写了不引入任何三方JAR的重试方法，在原始调用方法的外部包裹此方法即可，实现较为简单，仅作分享。

## 介绍

&ensp;&ensp;本例支持指定次数的重试，并且设定在多次重试的时间间隔。

```java
package com.pap.base.retry;

import java.util.concurrent.Callable;

public class RetryUtil {

    /**
     * 重试 工具类
     *
     * @param maxRetries  最大重试次数·
     * @param delayMillis 延迟时长
     * @param task        任务  () -> performOperation()    任务会抛出异常。
     * @param <T>
     * @return
     * @throws Exception
     */
    public static <T> T retry(int maxRetries, long delayMillis, Callable<T> task) throws Exception {
        int retryCount = 0;
        Exception lastException = null;

        while (retryCount < maxRetries) {
            try {
                T call = task.call();
                return call;
            } catch (Exception e) {
                lastException = e;
                System.out.println("Exception caught: " + e.getMessage());

                // 增加重试计数
                retryCount++;

                if (retryCount < maxRetries) {
                    // 等待一段时间后进行重试
                    waitBeforeRetry(delayMillis);
                } else {
                    throw lastException; // 抛出最后一个异常，达到最大重试次数
                }
            }
        }

        throw new IllegalStateException("Should not reach here");
    }

    private static void waitBeforeRetry(long delayMillis) {
        try {
            Thread.sleep(delayMillis);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    public static void main(String[] args) {
        try {
            String result = RetryUtil.retry(3, 1000, () -> performOperation());
            System.out.println("Operation result: " + result);
        } catch (Exception e) {
            System.out.println("Max retries reached: " + e.getMessage());
        }
    }

    private static String performOperation() throws Exception {
        // 模拟可能抛出异常的操作，这里可以抛出异常，模拟操作失败
        throw new Exception("Operation failed in : " + System.currentTimeMillis());
    }
}

```

## 总结

&ensp;&ensp;重试在各个框架中都有不同的实现，本例方法不一定完全适用所有场景，仅是一种最简单的实现。

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
