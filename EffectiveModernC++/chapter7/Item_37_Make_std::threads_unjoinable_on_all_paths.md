# Item 37: Make `std::threads` unjoinable on all paths

`std::thread` object has two states: *joinable* and *unjoinable*

*joinable* threads are corresponding to an underlying asynchronous thread of execution that:
* is/could be running
* blocked or waiting to be scheduled
* has run to completion

*unjoinable* threads are `std::thread` objects are not joinable
* default constructed `std::thread`
* std::thread objects that have been moved from
* std::thread objects that have been joined
* std::thread objects that have been detached
```c++
std::thread t = std::thread();
///more todo
```

If the destructor for a joinable thread is invoked, execution of the program is terminated. This equals to no call to `join()` at all, so that thread object out of scope die. resulting to terminated program.
```c++

```
In the following code we need to set priority of a thread so that we can not use `std::async` but we use `std::thread`.
```c++
#include <iostream>       
#include <thread>         
        
constexpr auto tenMillion = 10000000;

bool doWork(std::function<bool(int)> filter, int maxVal = tenMillion)
{
    std::vector<int> goodVals;
    
    std::thread t([&filter, maxVal, &goodVals]{
        for (auto i = 0; i <=maxVal; i++)
        {
            if (filter(i))
                goodVals.push_back(i);
        }
    })
        
    auto native_handle = t.native_handle();
    
    //set t's priority via native handle
    
    if(ConditionSatisfied())
    {
        t.join();
        doComputation(goodVals);
        return true;
    }
    return false;
}

int main() 
{
  return 0;
}
```
The problem is when `ConditionSatisfied()` returns `false`, or it throws an exception, `std::thread` object will be joinable when its destructor is called at the end of scope. This causes program execution to be terminated.

Solve this with RAII:
