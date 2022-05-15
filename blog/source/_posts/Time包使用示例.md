---
title: GO Time包使用示例 
date: 2022-05-14 21:46:06 
categories:
- GO语言 
tags:
- GO语言
---

# GO Time包使用示例

在GO语言中,时间相关的包为**time**

我们可以直接看go源码的src文件，里面有关于timer这个包的test代码

下面列举部分常用的time相关函数

## 定时器

##### time.After()

在时间结束后往写入通道

例子如下：

```
func ExampleAfter() {
	select {
	case m := <-c:
		handle(m)
	case <-time.After(10 * time.Second):
		fmt.Println("timed out")
	}
}
```

##### time.AfterFunc()

固定时间后执行函数

```
func TestAfterFunc(t *testing.T) {
	c := make(chan bool)
	var f func()
	fmt.Println("开始：" + time.Now().String())
	f = func() {
		fmt.Println("触发定时：" + time.Now().String())
		c <- true
	}

	time.AfterFunc(time.Second*10, f)
	<-c
}
```

```
开始：2022-05-14 22:53:18.706485 +0800 CST m=+0.000048501
触发定时：2022-05-14 22:53:28.707644 +0800 CST m=+10.001264751
```

##### time.Tick()

生成一个定时器，示例如下

```
c := time.Tick(5 * time.Second)
for next := range c {
	fmt.Printf("date：%v timeSteamp：%v\n", next.Format(time.RFC3339), next.Unix())
}
```

```
/*
date：2022-05-14T22:13:53+08:00 timeSteamp：1652537633
date：2022-05-14T22:13:58+08:00 timeSteamp：1652537638
date：2022-05-14T22:14:03+08:00 timeSteamp：1652537643
*/
```

这个例子很有意思，原因在于使用range来读取管道，而且并没有提供关闭这个通道的方法，文档提示它不能被GC回收，使用不当会导致内存泄漏

文档解释说：如果您的程序不需要释放关闭的情况下，使用这个方法是合理的

##### time.NewTicker()

相比刚刚的tick函数，这个ticker似乎有更完善的机制

提供**ticker.Stop**来释放资源

提供**ticker.Reset**来重置

官方示例如下：

```
	ticker := time.NewTicker(time.Second)
	defer ticker.Stop()
	done := make(chan bool)
	go func() {
		time.Sleep(10 * time.Second)
		done <- true
	}()
	for {
		select {
		case <-done:
			fmt.Println("Done!")
			return
		case t := <-ticker.C:
			fmt.Println("Current time: ", t)
		}
	}
```

## 定时器准确性

定时器实际上也是由CPU轮询触发的，因此必定会遭遇系统中断等情况，导致实际触发时间有差异

一般而言，目前认为定时器能做到毫秒级的准确性

需要注意的是go在1.14版本之后的实现，由每个P队列挂载时间堆来实现

而堆的结构，顶部节点为最小值，所有的定时器，实际上就是往堆里面不断写入到期时间，即顶部的到期时间最近，他到期了，才有可能到下一层节点上的元素到期

因此涉及到 堆的更新和查询

目前定时器使用的堆为四叉堆，目的是为了降低层级，查询时间复杂度为O(log4 N)，查询时间有相比二叉堆有一定提升

#### Parse 解析时间

官方的支持的时间解析格式并不是特别友好，建议使用carbon库来解决相关问题

#### TimeZone

默认解释的时间区为UTC时间，一般情况下，我们转换为字符串时，需要改为 **Asia/Shanghai**时区

```
	loc, _ := time.LoadLocation("Asia/Shanghai")
	const shortForm = "2006-Jan-02"
	t, _ = time.ParseInLocation(shortForm, "2012-Jul-09", loc)
	fmt.Println(t)
```

#### 时间间隔 （Duration）

##### 字符串解析 **time.ParseDuration**

```
//根据字符串解释为时间间隔
//格式 如 10h 1h10m10s 1µs 300ms
// Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h".
hours, _ := time.ParseDuration("10h")
fmt.Printf("There are %.0f seconds in %v.\n", hours.Seconds(), hours)
// There are 36000 seconds in 10h0m0s.
```

##### 计算时间间隔 

##### time.Since()

如在一些任务执行完毕以后，计算它的耗时

```
	start := time.Now()
	//doSomething
	time.Sleep(time.Second * 2)
	defer func() {
		timeSpent := time.Since(start)
		fmt.Println("consume time:" + timeSpent.String())
	}()
```

```
consume time:2.001047625s
```


##### time.Until() 

到某个时间的的间隔，负数表示在输入时间之前




## 总结

定时器是很常见的包组件，它的精度是大家需要留意的。
