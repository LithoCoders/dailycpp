# Item 37: Make `std::threads` unjoinable on all paths

`std::thread` objects have two states: *joinable* and *unjoinable*

A *joinable* thread is corresponding to an underlying asynchronous
thread of execution that:
* is or could be running, e.g., blocked or waiting to be scheduled
* has run to completion

*unjoinable* threads are `std::thread` objects are not joinable
* default constructed `std::thread`
* std::thread objects that have been moved from
* std::thread objects that have been joined
* std::thread objects that have been detached
```c++
std::thread t1 = std::thread(); //default ctor

std::thread t2 = std::thread([](){});
std::thread t3 = std::thread(std::move(t2)); //move ctor. t2 is unjoinable
```

If the destructor for a joinable thread is invoked, execution of the
program is terminated. This equals to no call to `join()` at all, so
that thread object out of scope die. resulting to terminated program.

```c++
#include <thread>

int main()
{
    std::thread t1 = std::thread([](){});

    return 0;
} // t1 goes out of scope, its dtor is called causes program terminated
/*Output
terminate called without an active exception

Aborted
*/
```
Beter code:

```c++
#include <thread>

int main()
{
    std::thread t1 = std::thread([](){});
    t1.join();
    return 0;
}
```
In the following code we need to set priority of a thread so that we
can not use `std::async` but we use `std::thread`.

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

    auto native_handle = t.native_handle(); //return implementation
defined underlying thread handle.

    //set t's priority via native handle
    //Unsuccessful to run example from
en.cppreference.com/w/cpp/thread/thread/native_handle

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
The problem is when `ConditionSatisfied()` returns `false`, or it
throws an exception, `std::thread` object `t` is still joinable
whereas its destructor is called at the end of scope. This causes
program execution to be terminated. This is how `std::thread` dtor is
designed. No implicit join nor implicit detach are implemented in
`std::thread::~thread`. The author argues because of performance
pitfall and debugging troubles.

Users have responsibilities to make `std::thread` object *unjoinable*
on every path of program before calling its dtor. It is difficult to
solve this because you need to consider all possibilities such as:
`return`, `goto`, exception handling, `continue`, `break`, or `if`
command as above.

The technique to have a common action along every path is putting it
in dtor, i.e, RAII.

```c++
#include <thread>
#include <iostream>

class ThreadRAII
{
  public:
    enum class DestructorAction { join, detach };
    
    //ctor accepts only rvalue of std::thread because we don't want to copy a thread object
    //order of ctor:init action first before thread since data members are set before running thread.
    ThreadRAII(std::thread&& t, DestructorAction a) : action(a), t(std::move(t)) {}
    
    //dtor checks if thread is joinable before calling join() or detach()
    //if thread is unjoinable, calling these two yields undefined behavior
    ~ThreadRAII(){
        if (t.joinable()){
            if (action == DestructorAction::join)
            {                
                std::cout << " about to joined ..\n";
                t.join();
            }
            else
            {
                std::cout << " about to detached ..\n";
                t.detach();            
            }
        }
    }
    
    //get function to acces to underlying std::thread. Like, smart pointers
    std::thread& get() { return t; }
    
  private:
    DestructorAction action;
    std::thread t;           
};

int main()
{
    ThreadRAII tRAII_join (std::thread([](){std::cout << std::this_thread::get_id() << std::endl;}),
                           ThreadRAII::DestructorAction::join);
    
    std::thread t_detach = std::thread([](){std::cout << std::this_thread::get_id() << std::endl;});
    ThreadRAII tRAII_detach (std::move(t_detach), 
                             ThreadRAII::DestructorAction::detach);
    
    std::cout << std::this_thread::get_id() << " about to end ..\n";
    return 0;
}

/*Output
140437722859328 about to end ..
 about to detached ..
 about to joined ..
140437696722688
140437705115392
*/
```
The output is chaotic but we know, there are three asynchronous threads (including `main`). Main thread calls dtor of `ThreadRAII` objects so that we see threads join or detach.

 If `get()` function of smart pointers give us the access to underlying raw pointer, `ThreadRAII` provides `get()` function to access to underlying `std::thread`.
 
 We can have the following code
 ```c++
    ThreadRAII tRAII_join (std::thread([](){std::cout << std::this_thread::get_id() << std::endl;}),
                           ThreadRAII::DestructorAction::join);   
    
    std::thread fooThread = std::move(tRAII_join.get());
    fooThread.join();  //We need to call join() or detach()    
 ```
 As such, dtor of `tRAII_join` is not called because data member `t` is not joinable.
 
 As a rule of three, it is nice to define move constructor and move assignment. We can ask compiler to generate these two for us.
 
 ```c++
#include <thread>
#include <iostream>

class ThreadRAII
{
  public:
    enum class DestructorAction { join, detach };
    ThreadRAII(std::thread&& t, DestructorAction a) : action(a), t(std::move(t)) {}
    
    ~ThreadRAII(){
        if (t.joinable()){
            if (action == DestructorAction::join)
            {                
                std::cout << " about to joined ..\n";
                t.join();
            }
            else
            {
                std::cout << " about to detached ..\n";
                t.detach();            
            }
        }
    }
    
    ThreadRAII(ThreadRAII&& tRAII) = default;
    ThreadRAII& operator=(ThreadRAII&& tRAII) = default;
    
    std::thread& get() { return t; }
    
  private:
    DestructorAction action;
    std::thread t;           
};

int main()
{
    ThreadRAII tRAII_join (std::thread([](){std::cout << std::this_thread::get_id() << std::endl;}),
                           ThreadRAII::DestructorAction::join);
    
    std::thread t_detach = std::thread([](){std::cout << std::this_thread::get_id() << std::endl;});
    ThreadRAII tRAII_detach (std::move(t_detach), 
                             ThreadRAII::DestructorAction::detach);
    
    std::thread t = std::move(tRAII_join.get());
    t.join();
    
    std::cout << std::this_thread::get_id() << " about to end ..\n";
    return 0;
}
 ```
 ## Things to remember:
 * Make `std::thread` objects unjoinable on all paths: by code manually(e.g., join()/detach() ) or by deploying `RAII`.
 * In RAII ctor, initialize `std::thread` at last.
 * join-on-destruction can lead to performance issue
 * detact-on-destruction can lead to undefined behavior
