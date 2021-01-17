---
title: JVM 调优案例
date: 2020-11-27 20:40:54
tags: JVM
---

## 命令行工具

```
jps -v: 查找java的进程，虚拟机进程状况工具

-XX 高级选项 针对开发
-XX: +PrintGC (+表示开启，-表示关闭)
-XX key=123

jinfo -flags [pid] 查看 VM的参数
jinfo -flag +PrintGC [pid] 运行时修改 VM的参数

jstat 统计一些信息：比如GC
jstat -gc [pid] 2000 10:显示垃圾回收情况, 每次隔2秒钟，统计10次

jmap 内存印象工具
jmap -heap
jmap -histo [pid]: 查找对象

jstack Java推栈跟踪工具
jstack [pid]
```

![](/images/jvm_opt/problem.png)


```
public class FullGCProblem {

	private static ScheduledThreadPoolExecutor executor 
	= new ScheduledThreadPoolExecutor(50, new ThreadPoolExecutor.DiscardOldestPolicy());

	public static void main(String[] args) throws Exception {
		// 50个线程
		executor.setMaximumPoolSize(50);
		while (true) {
			calc();
			Thread.sleep(100);
		}
	}

	// 多线程执行计算任务
	private static void calc() {
		List<UserInfo> taskList = getAllCardInfo();
		taskList.forEach(userInfo -> {
				executor.scheduledWithFixedDelay(() -> {
					userInfo.user();
				}, 2, 3, TimeUnit.SECONDS);
			});
	}

	// 模拟从数据库读取数据，返回
	private static List<UserInfo> getAllCardInfo() {
		List<UserInfo> taskList = new ArrayList<>();
		for (int i = 0; i < 100; i++) {
			UserInfo userInfo = new UserInfo();
			taskList.add(userInfo);
		}
		return taskList;
	}

	private static class UserInfo() {
		String name = "kai";
		int age = 18;
		BigDecimal money = new BigDecimal(99999);

		public void user() {
			// 业务逻辑什么都没做
		}
	}
}
```

CPU占用过高，垃圾回收期在疯狂工作
阿里开发手册 线程命名

