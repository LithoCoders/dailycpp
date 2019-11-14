
# Item 35 Prefer task based programming to thread based

One of the major changes that was incorporated into C++11 was the introduction of concurrency. This meant that devs can write 
multithreaded applications that have standard behavior accross platforms. Let's start by looking at what a thread is. According
to cplusplus.com : "A thread of execution is a sequence of instructions that can be executed concurrently with other such sequences in
multithreading environments, while sharing a same address space."

Let's have a quick look at a simple example that 
```c++
#include<iostream>
#include <thread>
#include <mutex>

int shared_variable = 100;
std::mutex mutex;

int worker(int thread_id)
{
    std::lock_guard<std::mutex> lockGuard(mutex);
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

In order to prevent access to the shared resource my multiple threads simultaneously, `std::mutex` is used. `std::lock_guard` 
object is used to lock the mutex during its construction and automatically releases it when it goes out of scope(i.e it's destructor is called).
Try to run this code without mutex multiple times and you will notice that each time, you will get a different result. This is because each
of the thread starts execution concurrently and they access the `std::cout` and shared variable simultaneously. This kind of
programming is referred to as thread based.

In this item, the author suggests the reader to prefer task based approach. In C++11, task based approach is accomplished by
the use of `std::async` objects. 

```c++
#include<iostream>
#include <thread>
#include <mutex>
#include <future>

int shared_variable = 100;
std::mutex mutex;

int worker(int thread_id)
{
    std::lock_guard<std::mutex> lockGuard(mutex);
    std::cout<<"thread"<<thread_id<<std::endl;
    shared_variable = shared_variable - 10;
    return 0;
}

int main()
{
 
    for(int i = 0; i<10; ++i)
    {
        auto return_value = std::async(std::launch::async, worker, i); //auto is deduced to std::future<int>
    }
    
     std::cout<<"shared_variable="<<shared_variable<<std::endl;   
    return 0;
}

```

One main difference is that it is possible to access the value returned by the `worker` function in the task based approach.
It is also possible in the thread based approach but it is not striaght forward. In the thread based approach, if the thread
throws an exception, then the program terminates via a call to `std::terminate`. However, in the task based approach it is possible to handle the exception.

```c++

#include<iostream>
#include <thread>

int worker()
{
    throw std::runtime_error("exception");
}

int main()
{
    try
    {
      std::thread w(worker);  
       w.join();
    }
    catch(...)   
    {
        std::cout<<"exception caught"<<std::endl; // code will not reach this point
    }
    
}
```

With `std::async`:

```c++
#include<iostream>
#include <future>

int worker()
{
    throw std::runtime_error("exception");
    return 0;
}

int main()
{
    try
    {
      auto futr = std::async(std::launch::async, worker);
      futr.get();
    }
    catch(const std::runtime_error& error)   
    {
        std::cout<<"exception caught"<<std::endl; // code will reach this point
    }
    
}
```

