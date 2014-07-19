---
layout: post
category: question
title: 关于vfork函数return和exit结果不同所引起的思考
tagline: by Sishuiliunian
tags: [ LinuxC]
---

## 问题描述
最近在重看《LinuxC编程实战》与《Unix环境高级编程》
在看到关于vfork这里的时候，出现了一些问题
以下为出现问题程序的代码

    enter code here 
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
    	switch(pid)
    	{
    		case 0:
    			i = 3;
    			while (i-- > 0)
    			{
	    			printf("Child process is running var = %d  globVar = %d\n",    var, globVar);
    				globVar++;
	    			var++;
	    			sleep(1);
	    		}
	    		printf("Child`s globVar = %d, var = %d\n",globVar,var);
	    		break;   //wrong
	    		//exit(0);   //right
	    		//_exit(0);
	    	case -1:
    			perror("Process creation failed\n");
    			exit(0);
    		default:
    			i = 5;
    			while (i-- > 0)
    			{
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
正如图中所示  我在子进程的循环结束之后使用break退出switch-case语句，此时如果程序的末尾以“exit(0)”作为结尾，那么程序的执行结果与书上所表述的吻合
![image](https://raw.githubusercontent.com/Gaoyuan0710/FAQ/gh-pages/images/The-different-between-returnAndexit-whenUsingVfork/2.png)
而当我将main函数末尾的exit(0)改为return 0时  程序出现了一些错误
![image](https://raw.githubusercontent.com/Gaoyuan0710/FAQ/gh-pages/images/The-different-between-returnAndexit-whenUsingVfork/3.png)
在显示调用exit后能够得到正确的执行结果，而通过return 0的隐式调用为什么是错误的？  想知道在子进程结束到父进程开始这段时间内到底做了哪些工作，为什么会出现以上的错误
《Unix环境高级编程》上讲：“如果子进程修改数据、进行函数调用、或者没有调用exec或exit就返回都可能会带来未知的结果” 
