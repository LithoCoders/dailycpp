                                            *Credits to ISLA*
There are two default capture modes in lambda: default by-reference capture `[&]` and default by-value capture `[=]`. `[&]` can lead to dangling reference whereas, `[=]` seems to fix the problem but you are not. In any case, the author advise to avoid both `[&]` and `[=]`. Instead, make captures explicitly for each variables lambda needs.

Considering a global container, which holds a list of functions. Each function captures a `divisor` from its surrounding (same function-scope or same class). The issue pops up when `divisor` can be dangled because it is out of its scope. 

# Problem with default by-reference capture
```c++
#include<vector>
#include<functional>
#include<iostream>

using FilterContainer = std::vector<std::function<bool(int)>>;
FilterContainer filters;

void addDivisorFilter()
{
    auto divisor = 2 ;
    filters.emplace_back(
                         [&] //default capture by reference
                          (int value) 
                            {std::cout << divisor << std::endl; return value % divisor == 0;}
                        );   
}

int main()
{
    addDivisorFilter();
    if(filters.size() > 0)
    {
        auto f= filters[0];
        std::cout << std::boolalpha << f(4);
    }
    return 0;        
}
//Output:
32767
false
```
In the above example, `filters` has a list of lambdas. Whereas all of them capture `divisor` by reference. At the call side in `main`, lamda has no knowledge of `divisor` as it ceases after its scope `addDivisorFilter()`. `divisor` takes a random number and thus our expectation, `4 % 2` is not met.

Even if we make capture `divisor` specifically, we still have the same problem with default capture by reference. However writting it down clearly `[&divisor]` raises a concern to see if `divisor` still exists when calling lambdas.
```c++
    filters.emplace_back(
                         [&divisor] //same problem as above
                          (int value)
                           {std::cout << divisor << std::endl; return value % divisor == 0;}
                        );   
```
Fix this problem with default capture by value
```c++
#include<vector>
#include<functional>
#include<iostream>

using FilterContainer = std::vector<std::function<bool(int)>>;
FilterContainer filters;

void addDivisorFilter()
{
    auto divisor = 2 ;
    filters.emplace_back(
                         [=] //default capture by value
                          (int value) 
                            {std::cout << divisor << std::endl; return value % divisor == 0;}
                        );   
}

int main()
{
    addDivisorFilter();
    if(filters.size() > 0)
    {
        auto f= filters[0];
        std::cout << std::boolalpha << f(4);
    }
    return 0;        
}
//Output
2
true
```
# Default by-value capture
In previous example, we have `divisor` just in the surrounding scope of lambda. Now if we have `divisor` as private data member of a class `Widget`.

```c++
#include<vector>
#include<functional>
#include<iostream>

using FilterContainer = std::vector<std::function<bool(int)>>;
FilterContainer filters;

class Widget {
    public:
        Widget(int d) : divisor(d){}
        void addFilter() const;
    private:
        int divisor;
};

void Widget::addFilter() const
{
    filters.emplace_back(
        [=](int value) { return value % divisor == 0; }
    );
}

int main()
{
    Widget w(2);
    w.addFilter();
    std::cout << std::boolalpha << filters[0](3);
    return 0;
}
//Output:
false

0
```
The above code compiled ok with C++14, but get a warning on C++2a even though it runs OK.
```c++
prog.cc: In lambda function:
prog.cc:19:9: warning: implicit capture of 'this' via '[=]' is deprecated in C++20 [-Wdeprecated]
   19 |         [=](int value) { return value % divisor == 0; }
      |         ^
prog.cc:19:9: note: add explicit 'this' or '*this' capture

false

0
```
As the error log mentioned, it implicitly capture `this`. So, replace default capture by value `[=]` with `[this]`:

```c++
void Widget::addFilter() const
{
    filters.emplace_back(
        [this](int value) { return value % divisor == 0; }
    );
}
```
Or the following code also works:
```c++
void Widget::addFilter() const
{
    auto currentObjectPtr = this;
    filters.emplace_back(
        [currentObjectPtr](int value) { return value % currentObjectPtr->divisor == 0; }
    );
}
```
Every non-static member has a `this` pointer. `Widget::addFilter()` can make use of `this` to access to `divisor`.

Smart pointer of `Widget`
```c++
#include<vector>
#include<functional>
#include<iostream>
#include<memory>

using FilterContainer = std::vector<std::function<bool(int)>>;
FilterContainer filters;

class Widget {
    public:
        Widget(int d) : divisor(d){}
        void addFilter() const;
    private:
        int divisor;
};

void Widget::addFilter() const
{
    filters.emplace_back(
        [this](int value) 
          { std::cout << this->divisor << std::endl; return value % this->divisor == 0; }
    );
}

void func()
{
    auto p = std::make_unique<Widget>(3);    
    p->addFilter();
}

int main()
{
    Widget w(2);
    w.addFilter();
    std::cout << std::boolalpha << filters[0](3) << std::endl;
    
    auto pw = std::make_unique<Widget>(3);    
    pw->addFilter();
    std::cout << std::boolalpha << filters[1](3) << std::endl;
    
    func();
    std::cout << std::boolalpha << filters[2](3) << std::endl; //filter holds dangling pointer
                                                               //because `p` inside func() destroyed
    return 0;
}
//Output:
2
false
3
true
0

Floating point exception
```
`w` and `pw` still exist after `return` in `main()`, but not `p` inside `func()`
So that, `this` of `p` does not exist anymore whereas `this` of `w` and `pw` still exist.

To solve this, we need to explicitly capture by value of `divisor` in C++14: 
```c++
void Widget::addFilter() const
{
    filters.emplace_back(
        [divisor = divisor](int value) 
          { std::cout << divisor << std::endl; return value % divisor == 0; }
    );
}
```
Final code we have:
```c++
#include<vector>
#include<functional>
#include<iostream>
#include<memory>

using FilterContainer = std::vector<std::function<bool(int)>>;
FilterContainer filters;

class Widget {
    public:
        Widget(int d) : divisor(d){}
        void addFilter() const;
    private:
        int divisor;
};

void Widget::addFilter() const
{
    filters.emplace_back(
        [divisor = divisor](int value) 
          { std::cout << divisor << std::endl; return value % divisor == 0; }
    );
}

void func()
{
    auto p = std::make_unique<Widget>(3);    
    p->addFilter();
}

int main()
{
    Widget w(2);
    w.addFilter();
    std::cout << std::boolalpha << filters[0](3) << std::endl;
    
    auto pw = std::make_unique<Widget>(3);    
    pw->addFilter();
    std::cout << std::boolalpha << filters[1](3) << std::endl;
    
    func();
    std::cout << std::boolalpha << filters[2](3) << std::endl; //filter holds dangling pointer
                                                               //because `p` inside func() destroyed
    return 0;
}
//Output:
2
false
3
true
3
true
```
