# 循序渐进C++并发编程

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

​			\#include <unistd.h>

​			\#include <pthread.h>



​			\#include <iostream>

​			\#include <thread>

​			using std::cout;

​			using std::endl;

​			using std::thread;



​			// 为了简便，声明成员为public以直接访问

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

​				int sum;

​			};



​			void* sum(void* args) {

​				ThreadContext* context = static_cast<ThreadContext*>(args);

​				for (int i = 0; i < context->data_len; ++i) {

​					context->sum += context->data[i];

​				}



​				return NULL;

​			}



​			int main(int argc, char *argv[]) {

​				int idata[100];

​				for (int i = 0; i < 100; ++i) {

​					idata[i] = i;

​				}



​				pthread_t thread_handler;

​				ThreadContext context_part1(idata, 50);

​				pthread_create(&thread_handler, NULL, sum,  (void*)&context_part1);



​				ThreadContext context_part2(idata+50, 50);

​				std::thread t1([&context_part2] () {

​					for (int i = 0; i < context_part2.data_len; ++i) {

​						context_part2.sum += context_part2.data[i];

​					}

​				});

​				t1.join();

​				pthread_join(thread_handler, NULL);

​				std::cout << "[child-thread] total sum = " << context_part1 + context_part2 << std::endl;



​				ThreadContext context_part(idata, 100);

​				sum((void*)&context_part);

​				std::cout << "[main-thread] total sum = " << context_part.sum << std::endl;

 

​				return 0;

​			}

​		```

​	将上述代码保存在文件test_thread.cc中，执行``` g++ -o test_thread test_thread.cc -std=c++14 -pthread```进行编译，再执行./test_thread，可以看到两个输出的结果均为4950。

​	为了使用gdb，可以增加-g编译选项增加debug信息，调试gdb ./test_thread，在子线程启动前后增加断点，执行i threads，可以看到线程列表，执行`thread 线程id`，可以执行bt观察该线程的上下文。





## 参考

《七周七并发模型》

http://blog.csdn.net/cxroom/article/details/9787155

