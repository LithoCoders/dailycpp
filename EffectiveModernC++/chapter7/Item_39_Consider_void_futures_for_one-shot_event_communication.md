# Item 39: Consider `void` futures for one-shot event communication

* Problem: Inter-thread communication. 
  * A task needs to inform an asynchronous task that an event has occured. 
  * The second task is blocked until the event has taken place.
  * An event could be: data structure has been initialized, computation has been completed, or a special sensor value has been detected. 
  
* Terminologies:
  * detecting task
  * reacting task

## Use condition variable
  Requires mutex
## Use flag
  Not blocking
## Use a combination of condition variable and flag
  Unnature, communication is somewhat stilted
## Use `std::promise<void>` and `void` futures
```c++
#include <thread>
#include <future>
#include <iostream>
std::promise<void> p;
void react() { std::cout << std::this_thread::get_id() << " Reacting to one-shot event..\n" ;};
void deact() 
{
    std::cout << std::this_thread::get_id() << " Detecting event ..\n" ;
    std::thread t([]
                  {
                      std::future<void> f = p.get_future();
                      f.wait();  // reacting task is blocked until promise is set. f.get() returns void.
                      react();
                  });
    p.set_value();
    t.join();
}

int main() {
    std::cout << std::this_thread::get_id() << " main ..\n" ;
    deact();
}
/*Output:
139665225992128 main ..
139665225992128 Detecting event ..
139665122731776 Reacting to one-shot event..
*/
```
Just re-write `std::thread` object using ctor to callable object `react()`

```c++
#include <thread>
#include <future>
#include <iostream>
std::promise<void> p;
void react(std::future<void> f) {
  f.wait();   // reacting task is blocked until promise is set
              // f.get() returns void.
  std::cout << std::this_thread::get_id() << " Reacting to one-shot event..\n" ;
};

void deact() 
{
    std::cout << std::this_thread::get_id() << " Detecting event ..\n" ;    
    std::thread t = std::thread(react, p.get_future());
    p.set_value();
    t.join();
}

int main() {
    std::cout << std::this_thread::get_id() << " main ..\n" ;
    deact();
}
/*Output
139798231173056 main ..
139798231173056 Detecting event ..
139798127912704 Reacting to one-shot event..
*/
```
Design ideas:
* Detecting task has `std::promise` object, writing to communication channel
* Reacting task has future object
* When detecting task see event happens, it *set*s the `std::promise`. In this case, it sets nothing. `p.set_value()`
* Reacting task waits on its future. This `wait` blocks the reacting task until the `std::promise` has been set
Both `std::promise` and futures (`std::future` and `std::shared_future`) are templates. The type of these templates are the type of data to be transmitted through the communication channel.
In this case, no data is transferred, thus it is `void`.

*shared state*s betwen `std::promise` and futures are dynamically allocated in heap. So, this design needs to consider the cost of heap-based allocation and dellocation.

This communication is *one-shot* communication, it can not be used repeatedly.



## Things to remember:
* Using `std::promise` and void futures solves the issues with condition variables, flags, or combincation of these. But this approach uses heap for shared states. This approach is for one-shot communication.
