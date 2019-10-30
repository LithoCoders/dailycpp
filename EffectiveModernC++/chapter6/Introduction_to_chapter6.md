Lambdas are convenience way to create function objects. 
It has a wide range of applications:
* STL `_if` algorithms such as `find_if`, `remove_if`, `count_if`
* Comparison functions such as `sort`, `nth_element`, `lower_bound`
* Custom deleter for `shared_ptr`, `unique_ptr`. Item 18 & 19
* Predicate for condition variables. Item 39
* On-the-fly call back functions
* interface adaptation functions. ?
* context-specific functions for one-off calls. ?

Vocabulary:
1. Lambda expression
```c++
#include <iostream>

int main()
{
    int x = 1; 
    int y = 1;
    
    auto l = 
      [=] () -> int { return x + y; };    //lambda expression
    std::cout << l();  
  
  
    return 0;
}
```
2. Closure class: each lambda causes compilers to generate a unique closure class at compile time.
For example, a closure class in the above example is `__lambda_9_7` below. We use C++ Insight, Standard C++ 2a
```c++
#include <iostream>

int main()
{
  int x = 1;
  int y = 1;
    
  class __lambda_9_7
  {
    public: 
      inline /*constexpr */ int operator()() const
      {
        return x + y;
      }
    private: 
      int x;
      int y;
    public:
      __lambda_9_7(int _x, int _y): x{_x}, y{_y}{}    
  };
  __lambda_9_7 l = __lambda_9_7{x, y};
  std::cout.operator<<(l.operator()());
  return 0;
}
```
This closure class *captures* `x` and `y` from the main function and makes them as private member data. It has one constructor taking `x` and `y` to initialize its private data member.
`operator()()` (same to functor ?) is the body of lambda.

3. Closure - runtime object of closure class. 
```c++
__lambda_9_7 l = __lambda_9_7{x, y};  // l is a closure
	                              // __lambda_9_7 is a closure class
```
We can have multiple closures from a closure type. 
```c++
#include <iostream>

int main()
{
    int x = 1; 
    int y = 1;
    
    auto l = 
      [=] () -> int { return x + y; };    //lambda expression
    auto l2 = l;
    auto l3 = l2;
    std::cout << l() << l2() << l3();    
    return 0;
}
//Output:
222
```
As needed for `l2` and `l3`, compiler also generates copy constructor.
```c++
#include <iostream>

int main()
{
  int x = 1;
  int y = 1;
    
  class __lambda_9_7
  {
    public: 
    inline /*constexpr */ int operator()() const
    {
      return x + y;
    }
    
    private: 
    int x;
    int y;
    public: 
    // inline /*constexpr */ __lambda_9_7(const __lambda_9_7 &) noexcept = default;
    __lambda_9_7(int _x, int _y)
    : x{_x}
    , y{_y}
    {}
    
  };
  
  __lambda_9_7 l = __lambda_9_7{x, y};
  __lambda_9_7 l2 = __lambda_9_7(l);
  __lambda_9_7 l3 = __lambda_9_7(l2);
  std::cout.operator<<(l.operator()());
  std::cout.operator<<(l2.operator()());
  std::cout.operator<<(l3.operator()());
  return 0;
}
```
