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
