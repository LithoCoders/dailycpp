# Questions are not in text book
1. Can you get launch policy when executing an async thread ? Get launch policy from future ?
2. get() vs. wait() vs. wait_for() ?
3. Can you schedule *deferred* launch policy to a thread, running on asynchrously ?
4. std::launch::async & std::launch::deferred ? std::launch::async | std::launch::deferred ?
5. Please create one entry about M&M rule. In chapter 7, we just use mutex without mutable.
6. `std::future` vs. `std::shared_future` ?
7. How to create a `std::thread` object ?
8. What are *joinable* and *unjoinable* ?
9. Example with `detach` ?
10. Can you create a `std::thread` with `std::function` ? to be merged with 7.

# Answers
## 7. How to create a `std::thread` object ?
We can create an object of `std::thread` from:
* a callable and its parameters
  * functor
  * pointers to a function
  * pointer to static member fuction
  * pointer to non-static member fuction
  * lambdas
* move constructor
```c++
#include <iostream>
#include <string>
#include <thread>
struct Functor
{
    void operator()(){ };        
};

void funct() {};

struct Widget
{
    static void static_funct(){};
    void funct(){};
};

int main()
{
    Functor f;
    std::thread t_functor = std::thread(f);
    
    std::thread t_function = std::thread(funct);    
    
    std::thread t_static_member_function = std::thread(&Widget::static_funct); //no need instantiated object   
    
    Widget w;
    std::thread t_nonstatic_member_function = std::thread(&Widget::funct, &w); //runs Widget::funct() on object w
    
    std::thread t_lambda = std::thread([](){});
    
    std::thread t = std::thread(f); // to be moved to t_move_ctor
    std::thread t_move_ctor = std::thread(std::move(t)); //t is no longer a thread
    
    //make sure all threads finish their jobs
    
    t_functor.join(); 
    t_function.join(); 
    t_static_member_function.join();    
    t_nonstatic_member_function.join();
    t_lambda.join();
    t_move_ctor.join(); //no need to join t
}
//main thread waits until all six threads are joined.
```
## 8. What are *joinable* and *unjoinable* ?
After threads are created, they are in *joinable* states. In above example, they can join the `main` thread or detach from main thread.
Opposite to this state is *unjoinable* state. Example, after joining or detaching, they are *unjoinable*.

```c++
#include <string>
#include <thread>
#include <iostream>

using namespace std::literals;
struct Functor
{
    void operator()(){ };        
};

void funct() {};

struct Widget
{
    static void static_funct(){};
    void funct(){};
};

int main()
{
    Functor f;
    std::thread t_functor = std::thread(f);
    
    std::thread t_function = std::thread(funct);    
    
    std::thread t_static_member_function = std::thread(&Widget::static_funct); //no need any instantiated object   
    
    Widget w;
    std::thread t_nonstatic_member_function = std::thread(&Widget::funct, &w); //runs Widget::funct() on object w
    
    std::thread t_lambda = std::thread([](){});
    
    std::thread t = std::thread(f); // to be moved to t_move_ctor
    std::thread t_move_ctor = std::thread(std::move(t)); //t is no longer a thread
    t_move_ctor.detach();
    
    //make sure all threads done their jobs
    
    if (t_functor.joinable()) t_functor.join(); 
    
    if (t_function.joinable()) t_function.join(); 
    
    if (t_static_member_function.joinable()) t_static_member_function.join();  
    
    if (t_nonstatic_member_function.joinable()) t_nonstatic_member_function.join();
        
    if (t_lambda.joinable())  t_lambda.join();
        
    if (t_move_ctor.joinable())
        t_move_ctor.join(); 
    else
        std::cout << "thread t_move_ctor is unjoinable";
}
```
Other reading: https://medium.com/@vgasparyan1995/let-me-detach-those-threads-for-you-2de014b26394

## 9. Example with `detach` ?
Detaching the thread from its calling thread. Both run independently from each other. Each of them releases resource when it ends execution. 

```c++
#include <iostream>       
#include <thread>         
#include <chrono>        

void pause_thread(int n) 
{
  std::this_thread::sleep_for (std::chrono::seconds(n));
  std::cout << "pause of " << n << " seconds ended\n";
}
 
int main() 
{
  std::cout << "Spawning and detaching 3 threads...\n";
  std::thread (pause_thread,1).detach(); //noname threads
  std::thread (pause_thread,2).detach(); //because not importatin
  std::thread (pause_thread,3).detach(); //to remember
  std::cout << "Done spawning threads.\n";

  std::cout << "(the main thread will now pause for 5 seconds)\n";
  pause_thread(5);
  return 0;
}
/*
Spawning and detaching 3 threads...
Done spawning threads.
(the main thread will now pause for 5 seconds)
pause of 1 seconds ended
pause of 2 seconds ended
pause of 3 seconds ended
pause of 5 seconds ended
*/
```
Source: http://www.cplusplus.com/reference/thread/thread/detach/
