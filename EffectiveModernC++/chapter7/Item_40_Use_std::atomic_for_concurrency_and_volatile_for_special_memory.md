# Item 40: Use `std::atomic` for concurrency, `volatile` for special memory

## `std::atomic`
We use `std::atomic` for data, accessed from multiple threads. 

* `std::atomic` is a template
```c++
#include <atomic>
#include <string>
#include <vector>
#include <utility>
#include <iostream>
struct Widget
{};

int main()
{
    std::atomic<int> ai;
    ai = 10;
    std::atomic<std::string*> as; 
    std::string s = "hallo";
    as = &s;
    std::cout << &s << std::endl ;
    std::cout << as;
    std::atomic<Widget*> aw;
}

```
* `std::atomic` behaves as if they were inside a mutex-protected critical session -> very efficient
  * fun fact with `std::cout << ai;`
  * RMW is also atomic
  * ordering
  ```c++
  int main()
{
    std::atomic<int> x;
    int y;
}
```
## `volatile`
* guarantees nothing about data race
* tell compiler thay they are dealing with memory that does not behave normally.
  * 'redundant loads' and 'dead stores'
  
  ```c++
    //redundant load
    volatile int x;
    auto y = x;
    y = x;  
  ```
  
    ```c++
    //not 'dead stores' anymore
    volatile int x = 1;
    x = 2;
  ```

## Mix usage between `std::atomic` and `volatile`
```c++
volatile std::atomic<int> vai; // operations on vai are
                                // atomic and can't be
                                 // optimized away
```

## Things to remember:
