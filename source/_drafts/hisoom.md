```java
2016-12-30 20:19:44,748 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:19:56,072 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:19:57,756 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:19:58,517 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:19:59,294 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:20:19,044 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:20:20,537 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:20:21,339 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:20:24,889 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:20:25,793 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:20:26,846 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:20:29,480 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:20:33,263 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:21:05,164 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (NioProcessor-2) - unhande exception:
java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:21:09,364 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:67} (EPC-1-worker-13) - unhande exception:
org.apache.mina.filter.codec.ProtocolEncoderException: java.lang.OutOfMemoryError: Java heap space
	at org.apache.mina.filter.codec.ProtocolCodecFilter.filterWrite(ProtocolCodecFilter.java:339)
	at org.apache.mina.core.filterchain.DefaultIoFilterChain.callPreviousFilterWrite(DefaultIoFilterChain.java:482)
	at org.apache.mina.core.filterchain.DefaultIoFilterChain.access$1400(DefaultIoFilterChain.java:47)
	at org.apache.mina.core.filterchain.DefaultIoFilterChain$EntryImpl$1.filterWrite(DefaultIoFilterChain.java:775)
	at org.apache.mina.core.filterchain.IoFilterAdapter.filterWrite(IoFilterAdapter.java:123)
	at org.apache.mina.core.filterchain.DefaultIoFilterChain.callPreviousFilterWrite(DefaultIoFilterChain.java:482)
	at org.apache.mina.core.filterchain.DefaultIoFilterChain.access$1400(DefaultIoFilterChain.java:47)
	at org.apache.mina.core.filterchain.DefaultIoFilterChain$EntryImpl$1.filterWrite(DefaultIoFilterChain.java:775)
	at org.apache.mina.core.filterchain.DefaultIoFilterChain$TailFilter.filterWrite(DefaultIoFilterChain.java:705)
	at org.apache.mina.core.filterchain.DefaultIoFilterChain.callPreviousFilterWrite(DefaultIoFilterChain.java:482)
	at org.apache.mina.core.filterchain.DefaultIoFilterChain.fireFilterWrite(DefaultIoFilterChain.java:475)
	at org.apache.mina.core.session.AbstractIoSession.write(AbstractIoSession.java:494)
	at org.apache.mina.core.session.AbstractIoSession.write(AbstractIoSession.java:439)
	at com.sangame.jjhcps.common.serverbase.server.online.base.OnlineSession.sendMsg(OnlineSession.java:178)
	at com.sangame.jjhcps.common.serverbase.server.online.base.BaseOnlineManager.sendMsg(BaseOnlineManager.java:533)
	at com.sangame.jjhcps.common.serverbase.server.online.base.BaseOnlineManager.respMsg(BaseOnlineManager.java:430)
	at com.sangame.jjhcps.common.serverbase.server.online.base.BaseOnlineManager.respMsg(BaseOnlineManager.java:456)
	at com.sangame.jjhcps.history.biz.t322.T322421.doBiz(T322421.java:90)
	at com.sangame.jjhcps.common.serverbase.event.BaseEvent.execute(BaseEvent.java:202)
	at com.sangame.jjhcps.common.epc.impl.BaseEpc$Task.run(BaseEpc.java:56)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
Caused by: java.lang.OutOfMemoryError: Java heap space
2016-12-30 20:21:26,357 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:65} (NioProcessor-4) - IOException
org.apache.mina.core.write.WriteToClosedSessionException
	at org.apache.mina.core.polling.AbstractPollingIoProcessor.clearWriteRequestQueue(AbstractPollingIoProcessor.java:638)
	at org.apache.mina.core.polling.AbstractPollingIoProcessor.removeNow(AbstractPollingIoProcessor.java:590)
	at org.apache.mina.core.polling.AbstractPollingIoProcessor.removeSessions(AbstractPollingIoProcessor.java:560)
	at org.apache.mina.core.polling.AbstractPollingIoProcessor.access$800(AbstractPollingIoProcessor.java:67)
	at org.apache.mina.core.polling.AbstractPollingIoProcessor$Processor.run(AbstractPollingIoProcessor.java:1132)
	at org.apache.mina.util.NamePreservingRunnable.run(NamePreservingRunnable.java:64)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
2016-12-30 20:21:26,358 [ERROR] com.sangame.phoenix.filter.DealExceptionFilter {DealExceptionFilter.java:65} (NioProcessor-4) - IOException
org.apache.mina.core.write.WriteToClosedSessionException
	at org.apache.mina.core.polling.AbstractPollingIoProcessor.clearWriteRequestQueue(AbstractPollingIoProcessor.java:638)
	at org.apache.mina.core.polling.AbstractPollingIoProcessor.removeNow(AbstractPollingIoProcessor.java:599)
	at org.apache.mina.core.polling.AbstractPollingIoProcessor.removeSessions(AbstractPollingIoProcessor.java:560)
	at org.apache.mina.core.polling.AbstractPollingIoProcessor.access$800(AbstractPollingIoProcessor.java:67)
	at org.apache.mina.core.polling.AbstractPollingIoProcessor$Processor.run(AbstractPollingIoProcessor.java:1132)
	at org.apache.mina.util.NamePreservingRunnable.run(NamePreservingRunnable.java:64)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
```

