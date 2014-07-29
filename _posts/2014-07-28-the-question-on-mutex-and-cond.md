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

