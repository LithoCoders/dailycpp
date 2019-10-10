This chapter focuses on *move semantics* and *perfect forwarding*. These concepts based on *rvalue reference*

What is the difference between a *lvalue* and *rvalue* ?
* rvalue: object is eligible for *move* operators while lvalue not
* rvalue: temporary object returned by a function, or literals
* lvalue: can be referred to by: name, pointer, or lvalue reference.
* lvalue: you can take its address while rvalue not.

Note: a parameter is always an lvalue.
```c++
void f(Widget&& w); //w is an lvalue.
```

# Move semantics with `std::move`
std::move vs. std::forward
* both do 'casting' to rvalue.
* `std::move` casts to rvalue without any condition.
* `std::forward` casts to rvalue with conditions.

```c++
#include <utility>      // std::move
#include <iostream>    
#include <vector>      
#include <string>      
int main () {
  std::string s1 = "s1";
  std::string s2 = "s2";
  std::vector<std::string> myvector;
  
  myvector.push_back (s1);                    // copies
  myvector.push_back (std::move(s2));         // moves
  
  std::cout << "myvector contains:";
  for (std::string& x:myvector) std::cout << ' ' << x;
  std::cout << std::endl;
   
  std::cout << "s1 is " << s1 << std::endl;
  std::cout << "Address of s2 is " << &s2 << " content is " << s2;
  return 0;
}
Start
myvector contains: s1 s2
s1 is s1
Address of s2 is 0x7ffd3cc3d4b0 content is 
0
Finish
```
`push_back()` function can take lvalue or rvalue. 

```c++
void push_back (const value_type& val);
void push_back (value_type&& val);
```

In the first call, it copies `s1` and put to the last element of `myvector`. `s1` still has its own value.
In the second call, `s2` is converted to rvalue then moved to the last item of `myvector`. `s2` still exists but unspecified state, we can not get `s2` value.

The following example shows you that xxxx
```c++
#include <utility>      // std::move
#include <iostream>    
#include <vector>      
#include <string>      
class Annotation{
    public:
        //explicit Annotation(std::string& text) : value(std::move(text)){}
        explicit Annotation(const std::string text) : value(std::move(text)){}
       
        std::string getValue() {return value;}
    private:
        std::string value;           
};

int main () {
  std::string s1 = "s1";
   
  Annotation a1(s1); 
  std::cout << "a1.value is " << a1.getValue() << std::endl;
  std::cout << "s1 is " << s1 << std::endl;
   
  return 0;
}
Start
a1.value is s1
s1 is s1
0
Finish
```
In this example, even though `std::move` casts `text` to rvalue but it still calls copying `text` to `value`. Because ctor taks a `const` value, so it can not be moved.
Two lessons:
1. Don't declare objects `const` if we want to perform move semantics from them.
2. `std::move` does not guarantee that rvalue it returns is eligible to be moved.
`std::move` guarantees to return you an rvalue.

# Perfect forwarding with `std::forward`
While `std::move` unconditionally casts its argument to an rvalue, `std::forward` **conditionally** casts its argument to an rvalue. 

So, when ? It is how `std::forward` is typically used, i.e., in function template taking universal reference parameters. 

```c++
#include <utility>    
#include <iostream>    

class Widget{
        
};

void process(const Widget& lvalWidget)
{
    std::cout << __PRETTY_FUNCTION__ << std::endl;
}
void process(Widget&& rvalWidget)
{
    std::cout << __PRETTY_FUNCTION__ << std::endl;
}
    
template<typename T>
void logAndProcess(T&& param)
{
    //do some logging 
    process(std::forward<T>(param));
}
    
int main () {
  Widget w;
  logAndProcess(w);                  //lvalue
  logAndProcess(std::move(w));       //rvalue
   
  return 0;
}
Start

void process(const Widget&)
void process(Widget&&)

0

Finish
```

`std::forward` casts to an rvalue if its argument was initialized with an rvalue.

Put the above code on C++ Insights we see both two functions are generated, one taking an rvalue, other taking an lvalue. If you remove one of the two `logAndProcess(w);` or `logAndProcess(std::move(w));`, only one function is generated from the template.

```c++
#include <utility>    
#include <iostream>    

class Widget
{
  public: 
  // inline constexpr Widget() noexcept = default;
  // inline constexpr Widget(const Widget &) = default;
  // inline constexpr Widget(Widget &&) = default;
};



void process(const Widget & lvalWidget)
{
  std::operator<<(std::cout, "void process(const Widget &)").operator<<(std::endl);
}

void process(Widget && rvalWidget)
{
  std::operator<<(std::cout, "void process(Widget &&)").operator<<(std::endl);
}

    
template<typename T>
void logAndProcess(T&& param)
{
    //do some logging 
    process(std::forward<T>(param));
}

/* First instantiated from: insights.cpp:26 */
#ifdef INSIGHTS_USE_TEMPLATE
template<>
void logAndProcess<Widget &>(Widget & param)
{
  process(std::forward<Widget &>(param));
}
#endif


/* First instantiated from: insights.cpp:27 */
#ifdef INSIGHTS_USE_TEMPLATE
template<>
void logAndProcess<Widget>(Widget && param)
{
  process(std::forward<Widget>(param));
}
#endif

    
int main()
{
  Widget w = Widget();
  logAndProcess(w);
  logAndProcess(std::move(w));
  return 0;
}
```
# Do you need both of them ?
The author mentioned that it is necessary to have both `std::move` and `std::forward` because these two do different actions. One prepares for a move, the other one passes/forwards an object to another function in a way that retains its original *lvalueness* or *rvalueness*
