#### 主线程Looper.loop()里面的无限循环为何不会导致应用卡死？

这里涉及到线程， 所以我们可以先聊一下进程和线程；

进程：

Android中每个APP运行前首先会创建一个进程，该进程由Zygote fork出来， 用于承载App四大组件的创建和运行；一般情况下一个App就运行在一个进程中，除非在Manifest中配置了Android：Process属性或者通过native代码fork进程。

线程：

线程是一段可执行的代码，也是计算机运行的最小单位，一个进程可以包含多个线程；

1.为何Looper.loop()会引入无限循环？

由于Android的主线程也是线程，线程一般执行完代码后就退出了，但Android肯定不希望主线程运行一小会儿就退出了， 那么如何能保证一直存活呢？简单做法就是可执行的代码可以一直执行下去， 无限循环便能保证可以一直执行下去。

2.那无限循环为何不会导致应用卡死呢？

这里涉及到Linux的pipe/epoll机制， 简单来讲就是在主线程的MessageQueque中没有消息的时候便会阻塞打牌looper的queue.next()方法，实际阻塞的方法是MessageQueue的nativePollOnce（native方法）方法，此时主线程会释放CPU资源进入休眠状态，直到下一个消息或者事务到来，再次唤醒主线程工作，所以说主线程休眠状态的时候不会占用CPU资源，就不会导致卡死或者什么的了。