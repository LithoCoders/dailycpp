
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
       w.join(); // either join or detach
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
    catch(...)   
    {
        std::cout<<"exception caught"<<std::endl; // code will reach this point
    }
    
}
```

One major difference that the author says between the task and the thread based approach is that the task based approach is more high level than the thread based approach. In the thread based approach, you will have to do manual thread management
but that is not the case in the thread based approach.

There are different kinds of threads:
* Hardware threads are ones that perform actual computation. It depends on the CPU(threads per core)
* Software threads are also called as OS threads which is managed by the OS to be executed on the hardware threads. Typically,you can have more SW threads than hardware threads. 
* `std::threads` are objects in a process and they act as handles to the OS software threads. If you try to create more threads than what the OS can handle, then an std::system_error is thrown even if the task is `noexcept`.

```c++
int work() noexcept;

std::thread t(work); throws if no more threads are available
```

In order to make sure that you do not overload the system, you need to make sure that all threads could be run. You could 
run out of threads that you can launch or you can have more threads that are ready to run. Typically, some of these issues
are handled by the OS's thread scheduler and often this is accomplised by means of time slicing. The ratio of the software to hardware threads vary from machine to machine and and the same solution would not work for different architectures.
So, one of the way to get rid of these issues is to use `std::async`. Thread management in such cases are taken care
by the C++ standard library implementation.
One advantage of `std::async` is that when it is launched in the default policy, it is not guaranteed that a new software thread is created. The function could be run on a different thread and this is managed by the thread scheduler.

In some cases, you could use `std::thread` to access the platform specific API, such as pthreads or Windows threads which 
offer more freedom such as thread affinities or priorities. `std::thread` offer `native_handle` member functions to accomplish this. Or you can use it in a case where you need to specifically optimize thread usage for a particular application.

In summary,

* `std::thread` does not offer any way to get return values from asynchronous tasks or functions and you cannot handle
exceptions thrown from these functions.
* In thread based programming, you have to do manual thread management, in task based, that is taken care by the C++
standard library implementation.
* Task based programming with default launch policy will mostly work out for you.

