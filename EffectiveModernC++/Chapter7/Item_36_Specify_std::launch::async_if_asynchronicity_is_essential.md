# Item 36: Specify std::launch::async if asynchronicity is essential

In previous item, we learn to prefer `std::async` to `std::thread`. And we know to prefer `std::async` over `std::thread`.
This item goes deeper in `std::async` launch policy.

There are two standard *launch policies*:
* **std::launch::async** - f runs asynchronously on different thread.
* **std::launch::deferred** - f is defered until `get` or `wait` is invoked on future - f runs synchronously - the caller is blocked until f finishes running.

???TODO: When you don't mention explicitly launch policy, the default one is applied. The default launch policy is neither of these. ???
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

So, we know the differences between *default launch policy* vs. *standard launch policies*
*default launch policy* has advantages: flexibility for thread management components of Standard library: thread creation, destruction, avoidance of overscription, and load balancing.

*default launch policy* has disadvantages:
* Not predict if f will run concurrently with t
* Not predict if f will run on different thread from t
* Not predict if f runs at all. It is not sure if `get()` or `wait()` is called on `future`.

```c++
#include <future>
#include <iostream>
using namespace std::literals;

void f()
{
    std::this_thread::sleep_for(1s); //sleep for one second then return
    std::cout << "thread " << std::this_thread::get_id() << std::endl;
}

int main()
{
    std::cout << "main " << std::this_thread::get_id() << std::endl;
    auto fut = std::async(f);  //default launch policy, can be std::launch::deferred

    while(fut.wait_for(100ms) != std::future_status::ready) //could run forever
    {
        std::cout << "hier..." << std::endl;
    }
    return 0;
}
//Output
main 140469374842816
hier...
hier...
hier...
hier...
hier...
hier...
hier...
hier...
hier...
thread 140469271582464
```
It seems, by luck the above thread is run asynchronously. In case, it is run synchronously, it can run forever.
