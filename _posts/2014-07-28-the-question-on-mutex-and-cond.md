---
layout: post
category: question
title: 关于线程条件变量和互斥锁的一点小疑问
tagline: by Reazon
tags: [ LinuxC]
---

## 问题描述
最近在看《LINUX系统编程手册》,在线程这里的互斥量和条件变量产生了疑问，下面理应是正确的做法，但是自己在思考时可能陷入了死胡同，所以觉得不对

	
	// 线程一代码
	pthread_mutex_lock(&mutex);
	if (A)
	pthread_cond_signal(&cond);
	pthread_mutex_unlock(&mutex);

	// 线程二代码
	pthread_mutex_lock(&mutex);
	while (！A)
	pthread_cond_wait(&cond, &mutex);
	pthread_mutex_unlock(&mutex);



## 思考疑点
1.如果线程一先获得锁，假设A为真，那么pthread_cond_signal执行时，没有线程在wait所以作废解锁退出，而线程二进入时由于!A所以直接解锁退出，此时是结果正确的
2.但是如果线程二先获得锁，假设A为假，那么pthread_cond_wait执行，陷入睡眠，但是此时，锁是不是仍在线程一手中，如果是的话，那线程二就会阻塞在pthread_mutex_lock(&mutex)这一句上，根本没有机会执行到if判断语句，此时两线程死锁

##代码检验
	#include <unistd.h>
	#include <assert.h>
	#include <stdio.h>
	#include <stdlib.h>
	#include <string.h>
	#include <pthread.h>

	pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
	pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

	int a = 1;

	void* fun1(void* arg)
	{
		pthread_mutex_lock(&mutex);
		if (a)
		{
			printf("fun1成功唤醒fun2\n");
		      	pthread_cond_signal(&cond);
		}
		pthread_mutex_unlock(&mutex);

		return NULL;
	}

	void* fun2(void* arg)
	{
		pthread_mutex_lock(&mutex);
		while (!a)
		{
			printf("fun2陷入休眠\n");
			pthread_cond_wait(&cond, &mutex);
		}
		pthread_mutex_unlock(&mutex);
	
		return NULL;
	}

	int main(int argc, char *argv[])
	{
		pthread_t thid1;
		pthread_t thid2;

		pthread_create(&thid1, NULL, fun1, NULL);
		pthread_create(&thid2, NULL, fun2, NULL);

		pthread_join(thid1, NULL);
		pthread_join(thid2, NULL);


		return EXIT_SUCCESS;
	}
当a = 1时，输出
fun1成功唤醒fun2
(但实际没有)
当a = 0时，输出
fun2陷入休眠

经检验和思考过程一致，那为什么那种模式是对的？或者说我应用的方式不对？
————————————————————

Qianyi:

嗯，你之前的理解都没有问题，但是注意下面这个细节：

	pthread_mutex_lock(&mutex);
	while (!a) {
		pthread_cond_wait(&cond, &mutex);
	}
	pthread_mutex_unlock(&mutex);

注意mutex其实是保证对这个条件判断的原子性的，也就是说**对这个条件的修改和判断必须是原子操作**。

而pthread_cond_wait容易让人误解的地方就是这里，pthread_cond_wait做的事情比较多。首先，pthread_cond_wait解开了那个mutex，然后等在了那个条件变量上（阻塞）。当signal被触发，离开pthread_cond_wait函数的时候，mutex又被自动加锁了。

**仔细理解这个过程，可以看到，pthread_cond_wait和我们自己的mutex一起配合，保证了我们对这个条件进行修改和判断的原子性。**

**为什么修改和判断这个条件需要原子性？这个不用解释吧？当两个线程同时操作一个变量的时候，时序可能会导致错误。那个银行存取钱的例子举了很多次了。**

如果我们假设pthread_cond_wait不会对锁进行操作（下面代码是错的），那么代码会写成下面这样：
	
	pthread_mutex_lock(&mutex);
	while (!a) {
		pthread_mutex_unlock(&mutex);
		pthread_cond_wait(&cond);
		pthread_mutex_lock(&mutex);
	}
	pthread_mutex_unlock(&mutex);

这样也能保证每次判断和修改都是原子的，但是每次都这么写，干脆mutex参数被加到了pthread_cond_wait函数，它代劳了。

**顺便说一句，原则上pthread_cond_signal(&cond);不需要加锁就可以发的，pthread_cond_signal(&cond);那片代码出现的mutex作用依旧是保证条件读写的原子性！**

**综上，pthread_cond_wait是一个实用主义的函数，当然，也容易搞糊涂初学者。**

无图无真相，下面分别是pthread_cond_wait等待前的解锁和成功返回时的解锁部分截图。

![image](https://raw.githubusercontent.com/hurley25/FAQ/gh-pages/the-question-on-mutex-and-cond/unlock.jpg)

![image](https://raw.githubusercontent.com/hurley25/FAQ/gh-pages/the-question-on-mutex-and-cond/lock.jpg)
