# Item 36: Specify std::launch::async if asynchronicity is essential

In previous item, we learn to prefer `std::async` to `std::thread`. This item goes deeper in `std::async` launch policy.

There are two standard *launch policies*:
* **std::launch::async** - f runs asynchronously on different thread.
* **std::launch::deferred** - f is defered until `get` or `wait` is invoked on future - f runs synchronously - the caller is blocked until f finishes running.
```c++
#include <future>
#include <iostream>
#include <vector>
#include <thread>

std::vector<std::string> vs;
std::mutex m;
void f(std::string s) 
{
    std::lock_guard<std::mutex> lock(m);    
    std::cout << "thread " << std::this_thread::get_id() << s << std::endl;
    vs.push_back(s);
};

int main()
{
    std::cout << "main " << std::this_thread::get_id() << std::endl;
    
    //C++14:If neither std::launch::async nor std::launch::deferred, 
    //nor any implementation-defined policy flag is set in policy, the behavior is undefined. 
    auto fut_default_policy = std::async(f, " fut_default_policy");  
    
    //execute f on new thread
    auto fut_async = std::async(std::launch::async, f, " fut_async");
    
    //does not spawn a new thread of execution. lazy evaluation
    auto fut_deferred = std::async(std::launch::deferred, f, " fut_deferred");    
    
    //indicates that the implementation may choose. 
    //This is the default option (when you don't specify one yourself). It can decide to run synchronously.    
    //https://stackoverflow.com/questions/30810305/confusion-about-threads-launched-by-stdasync-with-stdlaunchasync-parameter
    //run f either async or deferred
    auto fut_or = std::async(std::launch::async | std::launch::deferred, f, " fut_|");
    
    //run f with both async and deferred policy
    auto fut_and = std::async(std::launch::async & std::launch::deferred, f, " fut_&");    
    
    fut_deferred.get();
    fut_and.get();
    
    return 0;
}
//Output:
main 140161024485312
thread 140161024485312 fut_deferred
thread 140160912832256 fut_async
thread 140160904439552 fut_|
thread 140160921224960 fut_default_policy
thread 140161024485312 fut_&
```
We see `fut_deferred`, `fut_&` running on the same thread as `main`. It seems, Gcc implementation chooses `fut_|` to be run asynchronously whereas, `fut_&` to be run synchronously.

Putting this code on CppInsights C++2a, we only know all five threads inside `main` have the same type, namely `std::future<void>`. 

```c++
int main()
{
  std::operator<<(std::operator<<(std::cout, "main "), std::this_thread::get_id()).operator<<(std::endl);
  std::future<void> fut_default_policy = std::async(f, " fut_default_policy");
  std::future<void> fut_async = std::async(std::launch::async, f, " fut_async");
  std::future<void> fut_deferred = std::async(std::launch::deferred, f, " fut_deferred");
  std::future<void> fut_or = std::async(std::operator|(std::launch::async, std::launch::deferred), f, " fut_|");
  std::future<void> fut_and = std::async(std::operator&(std::launch::async, std::launch::deferred), f, " fut_&");
  fut_deferred.get();
  fut_and.get();
  return 0;
}
```

So, we know the differences between *default launch policy* vs. *standard launch policies*

1. *default launch policy* has advantages: flexibility for thread management components of Standard library: thread creation, destruction, avoidance of overscription, and load balancing.

2. *default launch policy* has disadvantages:
* Not predict if `f` will run concurrently with `main` thread, i.e, deferred launch.
* Not predict if `f` will run on different thread from `main` thread, i.e, async launch.
* Not predict if `f` runs at all. It is not sure if `get()` or `wait()` will be called on `future` along every path through the program

```c++
#include <future>
#include <iostream>

using namespace std::literals;

int f()
{
    std::this_thread::sleep_for(1s); //sleep for one second then return 0
    return 0;
}

int main()
{
    auto fut = std::async(f);  //default launch policy, can be std::launch::deferred

    std::future_status status = fut.wait_for(100ms);
    while(status != std::future_status::ready) //could run forever
    {
        if(status == std::future_status::deferred)
            std::cout << "Thread is deferred..." << std::endl;
         if(status == std::future_status::timeout)
            std::cout << "Timeout..." << std::endl;
    }
    return 0;
}
//Output
Timeout...
Timeout...
Timeout...
Timeout...
Timeout...
Timeout...
.....
Timeout...
Timeout...
Timeout...

File size limit exceeded
```
In this case, it seems thread run async. `fut.wait_for()` always returns `std::future_status::timeout` after 100ms !!! I don't know why thread is not executed and never be ready!!!

In a worse case, thread runs synchronously, `f` is deferred, `fut` runs with `async::launch::deferred` policy, `fut.wait_for()` always returns `std::future_status::deferred`, it never equals to `std::future_status::ready`, loop runs forever.

In general, if machine is overloaded: oversubcription or thread exhaustion, task may be most likely to be deferred. On the other hand, task may be most likely to be run asynchronously. But it seems, we don't have control over thread scheduling with default launch from `std::async`.

Fix by checking if task is deferred before entering the loop:
```c++
#include <future>
#include <iostream>
using namespace std::literals;

void f()
{
 std::this_thread::sleep_for(1s); //sleep for one second then return
}

int main()
{
    auto fut = std::async(f);  //default launch policy, can be std::launch::deferred
    
    if(fut.wait_for(0s) == std::future_status::deferred)
    {
            std::cout << "std::future_status::deferred..." << std::endl;
            fut.get();
    }
    else
    {
        while(fut.wait_for(100ms) != std::future_status::ready) //assuming f finishes
        {
            std::cout << "task is not deferred nor ready..." << std::endl;
        }
    }
    return 0;
}

//Output: Pitty that many tries show an async thread.
```

## Things to Remember
* The default launch policy mean both asynchronous and synchronous run.
* This flexibility leads to uncertainty which kind of run is it, thus don't know if you need to call `get()`.
* Thus, be explicit to say std::launch::async if asynchronous task execution is needed.
