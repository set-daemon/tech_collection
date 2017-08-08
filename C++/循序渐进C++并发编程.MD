# 循序渐进C++并发编程

*** 2017-08-07  set_daemon@126.com ***

​	并发基于计算机提供的能力，所以可以抛开语言来学习、分析和探讨，所谓的理论分析设计，不过相比于20、30年前，完成编程的成本极低，动手巩固所掌握的知识、学习验证新的知识还是非常有必要，所以还是先从C++开始这段旅程。



## 基础概念



### 并发及并行

​	参考网络上的一些资料，并发concurrency与并行parallel的区别不完全准确的理解：并发体现在排队容量上，并行体现在事务处理的速度上。

​	举个银行的例子，工作人员10人，每个人手头都有一个点钞器，每个点钞器可以容纳10万元厚度的钞票，由于要识别钞票的真假、损毁程度，机器工作比较慢，需要6分钟，现在有一个富豪A来了，有大约100万钞票需要清点，由于工作人员都空闲着，那就让每个工作人员领10万元厚度的钞票分别清点，把钞票塞进机器后，点钞器结束后会自动语音报告，结束后工作人员一起碰个头汇总下就完成富豪1的清点（这是并行，6分钟就可以完成原先1个小时的工作），不巧的是又一个富豪B来了，同样有100万左右的钞票要清点，在银行制度和点钞器的技术改进前，该富豪B要等机器完全空闲后才能开始清点，需要等12分钟，不过后来引入新的制度和技术后，只要点钞器有空闲，工作人员就可以在前后两个客户的钞票间用一个小挡板，把其它客户的钞票放入队尾就行，这时富豪B只需要等6分钟就行，但工作人员几乎是满负荷工作，同时处理几个客户的事务（这是并行，加快了事务的启动时间，提高了CPU的利用率，但并不能保证完成总时间是最快的，除非改进点钞器的识别算法加快点钞速度以及事务可并行拆解到更多CPU上执行）。





### 进程、线程、协程





### 互斥、信号量、消息队列、共享





### 事件驱动、消息驱动





## 初识线程 

​	在C++11标准以后，C++支持系统线程库pthread和语言级线程接口，下面代码实现两个线程计算求和运算：

​		```

​			\#include \<unistd.h>

​			\#include \<pthread.h>

​			\#include \<stdlib.h>

​			\#include \<sys/time.h>



​			\#include  \<iostream>

​			\#include \<thread>

​			using std::cout;

​			using std::endl;

​			using std::thread;



​			/\*

​			\* ThreadContext 线程Context，用于线程回调的输入参数

​			\*/

​			class ThreadContext {

​			public:

​				ThreadContext(int* _data, int _data_len) : data(_data), data_len(_data_len), sum(0) {}

​				ThreadContext() : ThreadContext(nullptr, 0) {}

​				int operator + (const ThreadContext& tc1) {

​					return sum + tc1.sum;

​				}

​			public:

​				int* data;

​				int data_len;

​				int sum;		      // 总和

​				double elapsed;  // 消耗毫秒数

​			};



​			/\*

​			\* Timing 监测类

​			\*/

​			class Timing {

​			public:

​				inline void start() {

​					gettimeofday(&tb, NULL);

​				}

​				inline void stop() {

​					gettimeofday(&te, NULL);

​				}

​				inline double elapsed() {

​					return (te.tv_sec - tb.tv_sec)*1000.0 + (te.tv_usec - tb.tv_usec)/1000.0;

​				}

​			private:

​				struct timeval tb;

​				struct timeval te;

​		 	};

​			/\*

​			\* sum 线程回调函数，也可以当作直接调用函数使用

​			\*/

​			void* sum(void* args) {

​				ThreadContext* context = static_cast<ThreadContext*>(args);

​				Timing tm;

​				tm.start();

​				for (int i = 0; i < context->data_len; ++i) {

​					context->sum += context->data[i];

​				}

​				tm.stop();

​				context->elapsed = tm.elapsed();			

​				return NULL;

​			}



​			int main(int argc, char *argv[]) {

​				int data_num = 1000;

​				if (argc > 1) {

​					data_num = atoi(argv[1]);

​				}

​				int *idata = new int[data_num];

​				for (int i = 0; i < data_num; ++i) {

​					idata[i] = i;

​				}



​				int half_num = data_num / 2;

​				// 计算事务A : 开始

​				Timing ta_tm;

​				ta_tm.start();

​				pthread_t thread_handler;

​				ThreadContext context_part1(idata, half_num);

​				pthread_create(&thread_handler, NULL, sum,  (void*)&context_part1);



​				ThreadContext context_part2(idata+half_num, half_num);

​				// C++11新特性

​				std::thread t1([&context_part2] () {

​					Timing local_tm;

​					local_tm.start();

​					for (int i = 0; i < context_part2.data_len; ++i) {

​						context_part2.sum += context_part2.data[i];

​					}

​					local_tm.stop();

​					context_part2.elapsed = local_tm.elapsed();

​				});

​				t1.join();

​				pthread_join(thread_handler, NULL);

​				// 计算事务A : 结束

​				ta_tm.stop();

​				

​				// 计算事务B: 开始

​				ThreadContext context_part(idata, data_num);

​				sum((void*)&context_part);

​				// 计算事务B: 结束



​				// 输出结果

​				std::cout << "case " << data_num  << "|" << context_part1.elapsed <<  "|" 

​						<< context_part2.elapsed << "|" << ta_tm.elapsed()  << "|" 

​						<<  context_part.elapsed << "|" <<  context_part1 + context_part2

​						 << "|" << context_part.sum << std::endl;

 				

​				delete idata;

​				return 0;

​			}

​		```

​	将上述代码保存在文件test_thread.cc中，执行``` g++ -g -o test_thread test_thread.cc -std=c++14 -pthread```进行编译，再执行./test_thread，可以看到事务A和事务B的求和结果相同。

​	为了观察线程运行情况，可以gdb调试，在适当的地方加断点，执行`i threads`查看当前线程列表，以及`t 线程id`进入线程上下文，`bt`查看线程调用栈。

​	

​	理论上分析，事务A由两个线程组成，而事务B在主线程中执行且进程内无更多其它线程，所以事务A的执行环境是包含主线程在内的三个线程定时调度，其消耗时间会受线程启动及调度影响，而事务B基本无调度影响（除操作系统进程间的调度外），其执行消耗时长即为真实cpu执行计算时长。

​	单独写一个测试脚本run_test_thread.sh：

​		```

​		\#!/bin/bash

​		if [ $# -lt 4 ]; then

​			echo "not enough parameters for $0"

​			exit -1

​		fi

​		beg_i=$1

​		end_i=$2

​		step=$3

​		case_times=$4

​		for ((i = beg_i; i <= end_i; i += step ))

​		do

​			for ((j = 0; j < case_times; ++j))

​			do

​				./test_thread ${i}

​			done

​		done

​		```

​	找一台比较干净的机器，设置适当的测试范围和步长、次数，执行./run_test_thread.sh，将数据重定向到文件中后再统计，可以看到一定的规律：

​		事务A的总时间 < 事务A的线程计算时间之和 占比约60.17%，小于A线程总和的4/5 占比约40.8%，小于A线程综合的3/5占比约7.06%；

​		事务A的总时间 < 事务B的总时间，占比约0.79%；

​	该结果说明事务A的两个线程是有一定并行化计算的，但由于计算是消耗CPU，无IO消耗，所以并发未体现出来。与预期事务A的总时间<=事务B总时间/2的结果相差太远，并行化不够，主要原因还是线程未被分派到单独的CPU上而被调度导致。



## 参考

《七周七并发模型》

http://blog.csdn.net/cxroom/article/details/9787155

