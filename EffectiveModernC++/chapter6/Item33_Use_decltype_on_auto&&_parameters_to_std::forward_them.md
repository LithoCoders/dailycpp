# Generic lambdas

Let us revise the syntax of lambda:
```c++
 1   2   3         4        
[=] () throw() -> int
{                              5
                int n;
                return n;
}
```
Number 2 is parameter list inside parenthesis. When we have `auto` in this parenthesis, we call *generic lambdas*.

Let us consider the following simple example:
```c++
#include <iostream>
#include <functional>


int main()
{
  auto c = [] (auto x) { return x > 0; };
  
  std::cout << std::boolalpha << c(-1);
  
  return 0;
}
//Output:
false
```
C++ Insight C++2a shows the compiler generate `operator()` as a template function and its instantiation when `c(-1)` is called.
```c++
#include <iostream>
#include <functional>


int main()
{
    
  class __lambda_7_12
  {
    public: 
    template<class type_parameter_0_0>
    inline /*constexpr */ auto operator()(type_parameter_0_0 x) const
    {
      return x > 0;
    }
    
    #ifdef INSIGHTS_USE_TEMPLATE
    template<>
    inline /*constexpr */ bool operator()(int x) const
    {
      return x > 0;
    }
    #endif
    
    private: 
    template<class type_parameter_0_0>
    static inline auto __invoke(type_parameter_0_0 x)
    {
      return x > 0;
    }
    
    public:
    // /*constexpr */ __lambda_7_12() = default;
    
  };
  
  __lambda_7_12 c = __lambda_7_12{};
  std::cout.operator<<(std::boolalpha).operator<<(c.operator()(-1));
  return 0;
}
```
The template class and its instantiation are:
```c++
    template<class type_parameter_0_0>
    inline /*constexpr */ auto operator()(type_parameter_0_0 x) const
    {
      return x > 0;
    }
    
    #ifdef INSIGHTS_USE_TEMPLATE
    template<>
    inline /*constexpr */ bool operator()(int x) const
    {
      return x > 0;
    }
    #endif
```
# Generic lambdas + perfect forwarding
Let us consider a more complex example where in lambda body, we make a call to other `func()`
```c++
auto c = [] (auto x) { func(x); };
```

`func` can treat `x` differently: rvalue or lvalue. We can solve this by:
* perfect-forwarding `x` to `func`
* `x` is univeral reference

```c++
void func(??? x);  //we want to make a generic `func`
auto c = [] 
           (auto&& x)                             //x is a universal reference
               { func( std::forward<???>(x) ) }   //perfect-forwarding to func
```
We don't know type of `x` yet. Let us break it down. 

First, considering implementation of `std::forward` in C++14

```c++
template<typename T> 
T&& forward(remove_reference_t<T>& param)    //conditional casting to rvalue
{                                            //by relying on reference collapsing
    return static_cast<T&&>(param);        
}
```
 `x` is universal reference. It can be lvalue reference or rvalue reference.

** Case 1: `x` is a lvalue reference `Widget`. Replace everywhere `T` with `Widget`. No reference collapsing happens here.

```c++
Widget&& forward(Widget& param)    
{                                            
    return static_cast<Widget&&>(param);        
}
```
** Case 2: `x` is a rvalue reference `Widget&&`. Replace everywhere `T` with `Widget&&`. Reference collapsing happens here.
```c++
Widget&& && forward(remove_reference_t<Widget&&>& param)    //conditional casting to rvalue
{                                            //by relying on reference collapsing
    return static_cast<Widget&& &&>(param);        
}

//Reference collapsing
Widget&& forward(Widget& param)    //conditional casting to rvalue
{                                            //by relying on reference collapsing
    return static_cast<Widget&&>(param);        
}
```

"If you compare this instantiation with the one that results when std::forward is called with T set to Widget, you’ll see that they’re identical."

"decltype(x) yields an rvalue reference type when an rvalue is passed as an argument to our lambda’s parameter x."

```c++
auto c = [] 
           (auto&& x)                                     //still. x is a universal reference
               { func( std::forward<decltype(x)>(x) ) }   //perfect-forwarding to func
```
# Generic lambdas + perfect forwarding + variadic
Add more dots to make it support variadic
```c++
auto c = [] 
           (auto&&... xs)                                     //still. x is a universal reference
               { func( std::forward<decltype(xs)>(xs)... ) }   //perfect-forwarding to func
```

In short, the author advises to use `decltype` to perfect-forward `auto&&` parameters.
