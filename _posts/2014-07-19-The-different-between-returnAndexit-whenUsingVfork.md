---
layout: post
category: question
title: 关于vfork函数return和exit结果不同所引起的思考
tagline: by Sishuiliunian
tags: [ LinuxC]
---

## 问题描述
最近在重看《LinuxC编程实战》与《Unix环境高级编程》，在看到关于vfork这里的时候，出现了一些问题。

以下为出现问题程序的代码：

	// enter code here 
	#include <unistd.h>
	#include <stdio.h>
	#include <stdlib.h>
	#include <sys/types.h>
	
	int globVar = 1;
	
	int main(int argc, char *argv[])
	{
		pid_t pid;
		int var = 1, i;
		printf("fork is diffirent with vfork \n");
		//pid = fork();
		pid = vfork();
		switch(pid) {
			case 0:
				i = 3;
				while (i-- > 0) {
					printf("Child process is running var = %d globVar = %d\n", var, globVar);
					globVar++;
					var++;
					sleep(1);
				}
				printf("Child`s globVar = %d, var = %d\n",globVar,var);
				break;       //wrong
				//exit(0);   //right
				//_exit(0);
			case -1:
				perror("Process creation failed\n");
				exit(0);
				default:
				i = 5;
				while (i-- > 0) {
					printf("Parent process is running var = %d globVar = %d\n",    var , globVar);
					globVar++;
					var++;
					sleep(1);
				}
				printf("Parent globVar = %d, var = %d\n", globVar, var);
				exit(0);
			break;
 		}
		printf("var = %d\n",var);
		return 0;
		//exit(0);
	}

## 讨论结果

vfork()函数在《LinuxC编程实战》中说道：“用vfork创建的子进程共享父进程的地址空间；子进程对地址空间中任何数据的修改同样为父进程所见”，“vfork保证子进程先执行，在子进程调用exit或exec之前父进程处于阻塞等待状态”

![image](https://raw.githubusercontent.com/Gaoyuan0710/FAQ/gh-pages/images/The-different-between-returnAndexit-whenUsingVfork/1.png)

正如图中所示  我在子进程的循环结束之后使用break退出switch-case语句，此时如果程序的末尾以“exit(0)”作为结尾，那么程序的执行结果与书上所表述的吻合。

![image](https://raw.githubusercontent.com/Gaoyuan0710/FAQ/gh-pages/images/The-different-between-returnAndexit-whenUsingVfork/2.png)

而当我将main函数末尾的exit(0)改为return 0时，程序出现了一些错误。

![image](https://raw.githubusercontent.com/Gaoyuan0710/FAQ/gh-pages/images/The-different-between-returnAndexit-whenUsingVfork/3.png)

在显示调用exit后能够得到正确的执行结果，而通过return 0的隐式调用为什么是错误的？  想知道在子进程结束到父进程开始这段时间内到底做了哪些工作，为什么会出现以上的错误。

P.S. 《Unix环境高级编程》上讲：“如果子进程修改数据、进行函数调用、或者没有调用exec或exit就返回都可能会带来未知的结果”。

————————————————————

Qianyi：

我说一下我的理解，首先我介绍一下vfork函数出现的原因：

在早期的时候，fork函数用于创建子进程，会**完全的**复制其父进程的完整的地址空间的内存页来为子进程创建空间。但是这么做的缺陷很明显，会有很多没有必要的复制产生，而且对于像shell的这样的程序而言，当我们敲下一个命令之后，会立即fork出子进程并且调用exec来为其加载代码和创建新的内存空间。早期的fork函数在这一点上造成了很大的效率问题。

于是，便有了vfork函数，vfork并不复制父进程的地址空间，而是**直接借用**原来父进程的地址空间里。想必你也看出来了，此时内核必须挂起父进程，不然的话两个进程运行在同一个地址空间里不出问题才怪。**内核保证vfork调用后，子进程先于父进程执行就是因为这个。**对于shell来说，vfork一下，挂起父进程，子进程临时借用父进程的地址空间（主要是代码段）执行一下exec族的函数，内核会立即为其加载新的地址空间，然后允许其父进程继续运行。这样冲突不久解决了么？ vfork函数甚至就是专门用来解决shell这种fork后需要理解exec加载新的程序的需求的。

**那么，如果我们不这么做，会发生什么事情？**

首先，共享整个地址空间意味着什么？**子进程对内存的所有修改对之后运行的父进程可见！！试想一下你和别人公用一个宿舍，你出去之后舍友搬动了一个椅子也走了，你再次回来的时候自然是你室友搬动后的样子。

然后，我们再来讨论一下在main函数里return和调用exit的区别。main里面return返回到什么地方？答案是库函数调用我们main函数的地方。大家没意见吧？其实我们在main函数里返回之后，库代码也是最终调用exit族的函数退出的。那么区别何在？看下面的示例：

	void glibc_某函数() {
		int ret = main();
		// do something...
		exit(ret);
	}
	
大家看到了什么？子进程main先返回了，库函数做了一些东西之后也调用了main退出了。**但是！！请注意栈的变化！！main返回后做了些事情，在原来main所在的位置调用了exit函数！！！**意味着main原先的局部变量存储很可能被破坏！那么，子进程推出后，父进程再次执行的时候...悲剧产生了。谁也不知道此时的栈被子进程破坏成啥样了，所以文档说**未知的结果**。不同的发行版和glibc库的版本完全就不一样了，对的话算运气好，不对也很正常。（全局变量存储区距离栈比较远，一般可以幸免）为什么直接调用exit不会出错？因为exit在main里调用，exit的栈临时空间在main之下，自然不会错。

理解了吗？顺带说一下自从fork实现了**写时拷贝**之后，vfork彻底**弃用**，所以别用vfork了，毕竟两个程序公用地址空间而且都可以读写太危险了！

至于exit和_exit的区别？大家会google的对吧？

下面是vfork的main文档。虽然没解释原因，但是man文档清楚地说明了应该立即调用_exit或者exec族的函数（离开暂时借用的父进程的空间）。

Linux Description

  vfork(), just like fork(2), creates a child process of the calling process.  For details and return value and errors, see fork(2).
  
  vfork()  is  a special case of clone(2).  It is used to create new processes without copying the page tables of the parent process.  It may be useful in performance-sensitive applications where a child is created which then immediately issues an execve(2).

