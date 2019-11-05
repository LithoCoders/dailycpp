# Capture modes: by-reference and by-value
```c++
#include <iostream>
#include <cstdlib>

int main()
{
    int x = 1; 
    int y = 1;
    std::cout << "Outside lambdas &x : " << &x << std::endl;
    auto lCaptureByValue = [=]           //default by-value capture for both x and y
                               () -> void
    {
        std::cout << "Inside lCaptureByValue &x : " << &x << std::endl;        
        //even though x, y are captured by value
        //we can not modify x & y here. See explanation below.
    };    
    lCaptureByValue();
    std::cout << x << "  " << y << std::endl;
    
    auto lCaptureByRef = [&]               //default by-ref capture for both x and y
                             () -> bool    
    {        
        std::cout << "Inside lCaptureByRef &x : " << &x << std::endl;        
        x--;        
        y--;        
        return (x + y) > 0;
    };
    std::cout << std::boolalpha << lCaptureByRef() << " " << x << "  " << y << std::endl;
    
    auto lCaptureByRefAndValue = [&x, y] () -> void  //[&x, =] does not work!!
    {    
        std::cout << "Inside lCaptureByRefAndValue &x : " << &x << std::endl;        
        x--;
    };
    lCaptureByRefAndValue();
    std::cout << x << "  " << y << std::endl;
    
    return 0;
}

//Output:
Outside lambdas &x : 0x7ffdbcb206fc
Inside lCaptureByValue &x : 0x7ffdbcb206f4
1  1
Inside lCaptureByRef &x : 0x7ffdbcb206fc
false 0  0
Inside lCaptureByRefAndValue &x : 0x7ffdbcb206fc
-1  0
```

# `operator()` of the generated functor
As we noticed with `lCaptureByValue` lambda, we can not modify `x` and `y`. This is because `operator()` of the generated functor is `const`.
Let us consider the generated code of `lCaptureByValue` with C++ Insight
```c++
  class __lambda_9_28
  {
    public: 
        inline /*constexpr */ void operator()() const   //this is a `const` function
        {
            ...
        }
    
    private: 
        int x;
    
    public:
        __lambda_9_28(int _x) : x{_x} {}    
  };
```
As we see, by default, `operator()` is a `const` thus it can not modify data member `x` or we say, objects captured by value in lambda are immutable by default. To modify `x` inside lambda, we need to use `mutable` keyword:
```c++
#include <iostream>
#include <cstdlib>

int main()
{
    int x = 1; 
    int y = 1;
    std::cout << "Outside lambdas &x : " << &x << std::endl;
    auto lCaptureByValue = [=] () mutable -> void
    {
        x++;
        std::cout << "Inside lambdas &x : " << &x << std::endl;
        std::cout << x << std::endl;
    };    
    lCaptureByValue();
    std::cout << x << "  " << y << std::endl;
      
    return 0;
}
//Output:
Outside lambdas &x : 0x7fff4da209f8
Inside lambdas &x : 0x7fff4da209f4
2
1  1
```
C++ Insight shows generated closure class
```c++
class __lambda_9_28
  {
    public: 
        inline /*constexpr */ void operator()()  //No longer `const`
            {
                x++;
                ...
            }
    private: 
        int x;
    
    public:
         __lambda_9_28(int _x) : x{_x} {}    
  };
```
As we see, `operator()` is no longer `const` so, we can change captured `x`.
How about by-reference capture ? 
```c++
#include <iostream>
#include <cstdlib>

int main()
{
    int x = 1; 
    int y = 1;
    auto lCaptureByRef = [&] () -> bool    //capture-by-ref for both x and y
    {      
        x--;        
        y--;        
        return (x + y) > 0;
    };
    
    return 0;
}
```
The generated closure from C++ Insight shows that by default `operator()` still `const` but closure class holds private references to `x` and `y`. So, we can modify `x` and `y` inside lambda body.
```c++
class __lambda_8_26
  {
    public: 
        inline /*constexpr */ bool operator()() const
        {
            x--;
            y--;
            return (x + y) > 0;
        }
    
    private: 
        int & x;
        int & y;
    
    public:
        __lambda_8_26(int & _x, int & _y) : x{_x}, y{_y}  {}
```

Source for mutable lambdas: https://mayankj08.github.io/2017/08/06/Mutable-Lambdas-In-C++/
