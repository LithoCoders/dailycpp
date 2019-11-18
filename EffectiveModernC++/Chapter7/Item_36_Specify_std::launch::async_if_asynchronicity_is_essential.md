In previous item, we learn to prefer `std::async` to `std::thread`. There are two *launch policies*:
* **std::launch::async** - f runs asynchronously.
* **std::launch::deferred** - f is defered until `get` or `wait` is invoked on future - f run synchronously - caller blocked until f finishes running.

When you don't mention explicitly launch policy, the default one is applied. The default launch policy is neither of these. 
```c++
#include <future>
#include <iostream>
 
void f() { std::cout << "Hello" << std::endl; };

int main()
{
    auto fut_undefined = std::async(std::launch::async, f);
    auto fut_async = std::async(std::launch::async, f);
    auto fut_deferred = std::async(std::launch::deferred, f);
    
    auto fut2 = std::async(std::launch::async | std::launch::deferred, f); // dont know what is this?
    
    fut_deferred.get();
    return 0;
}
//Output:
HelloHello

Hello
Hello

0

```

Advantage: flexibility for thread management components of Standard library: thread creation, destruction, avoidance of overscription, and load balancing.
Disavantage:
-Not predict if f will run concurrently with t
-Not predict if f will run on different thread from t
-Not predict if f runs at all.

```c++
using namespace std::literals;

void f()
{
 std::this_thread::sleep_for(1s); //sleep for one second then return
}

auto fut = std::async(f);                        //default launch policy, can be std::launch::deferred

while(fut.wait_for(100ms) != std::future_status::ready) //could run forever
{
 //do things
}
```
