
# Item 35 Prefer task based programming to thread based

One of the major changes that was incorporated into C++11 was the introduction of concurrency. This meant that devs can write 
multithreaded applications that have standard behavior accross platforms. Let's start by looking at what a thread is. According
to cplusplus.com : "A thread of execution is a sequence of instructions that can be executed concurrently with other such sequences in
multithreading environments, while sharing a same address space."

Let's have a quick look at a simple example that 
```c++
#include<iostream>
#include <thread>

int shared_variable = 100;

int worker(int thread_id)
{
    std::cout<<"thread"<<thread_id<<std::endl;
    shared_variable = shared_variable - 10;
    return 0;
}

int main()
{
    std::thread threads[10];
    
    for(int i = 0; i<10; ++i)
    {
        threads[i] = std::thread(worker, i);
    }
    
    for(int i = 0; i<10; ++i)
    {
        threads[i].join();
    }
     std::cout<<"shared_variable="<<shared_variable<<std::endl;   
    return 0;
}
```

Try to run this code multiple times and you will notice that each time, you will get a different result.
