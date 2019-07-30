# Item 8 - Prefer nullptr to 0 and NULL

Note: Contains code snippet from Effective Modern C++ by Scott Meyers


The author suggests to use `nullptr` in place of NULL or 0 to prevent the follwing issues:

1. To prevent overloading on pointer to integer types
2. To Prevent ambiguity

```C++
NULL -> either int or long depending on implementation
0 -> int
nullptr -> std::nullptr_t -> implicitly converts to all raw pointer types -> not an integral type
std::nullptr_t -> nullptr -> circular definition
```

example 1.1 with only one function taking void*
```C++
void f(void*)
{
    std::cout << "f takes void*" << std::endl;
    return;
}
    
int main()
{
    f(0);
    f(NULL);
    f(nullptr);
  	return 0;    
}
Output:
f takes void*
f takes void*
f takes void*
```
example 1.2 with only two overloaded functions
```C++
void f(int)
{
    std::cout << "f takes int" << std::endl;
    return;
}

void f(void*)
{
    std::cout << "f takes void*" << std::endl;
    return;
}
    
int main()
{
    f(0); // fine. "f takes int"
    f(NULL); // NOK because of ambigous overloaded function or might not compile, may call f(int)
    f(nullptr); // fine. "f takes void*"
  	return 0;    
}

prog.cc: In function 'int main()':
prog.cc:20:11: error: call of overloaded 'f(NULL)' is ambiguous
   20 |     f(NULL);
      |           ^
prog.cc:5:6: note: candidate: 'void f(int)'
    5 | void f(int)
      |      ^
prog.cc:11:6: note: candidate: 'void f(void*)'
   11 | void f(void*)
      |      ^

```
example 2 auto deduction
```C++
auto result = findRecord( /* arguments */ );

if(result == 0){} // unclear what findRecord returns

if(result == nullptr){} //result must be a ptr type

```
example 3.1 Casting NULL and 0 to shared_ptr and unique_ptr
```c++
#include<iostream>
#include<memory>
#include<mutex>

std::mutex m1, m2, m3;
using MuxGuard = std::lock_guard<std::mutex>;

class Widget
{
    public:
        void print()
        {
            std::cout << "I am a Widget" << std::endl;            
        }
};

// code duplication - mutex lock
int f1(std::shared_ptr<Widget> spw)
{
    MuxGuard g(m1); // lock mutex for f1
    spw->print();
    return 0;
}
double f2(std::unique_ptr<Widget> upw)
{
    MuxGuard g(m2); // lock mutex for f2
    upw->print();
    return 0.0;
}

bool f3(Widget* pw)
{
    MuxGuard g(m3); // lock mutex for f3
    pw->print();
    return 0;
}


int main()
{
  auto result = f1(NULL); // work
  auto result2 = f2(0); // work as well
  auto result3 = f3(nullptr); //ditto
  std::cout << result << std::endl;
  std::cout << result2 << std::endl;
  std::cout << result3 << std::endl;
}
```

The above implementation from the compiler is OK. Why does it work? Because the compiler does it's job for you ;)

Intermediate compiler output from cppinsights.io for the above impl(C++11):

```
int main()
{
  int result = f1(std::shared_ptr<Widget>(NULL)); //NULL is casted to the respective shared_ptr type
  double result2 = f2(std::unique_ptr<Widget, std::default_delete<Widget> >(0)); //0 is casted to unique_ptr type.
  bool result3 = f3(nullptr);
}
```
example 3.2. Using function template to remove code duplicates
```c++
#include<iostream>
#include<memory>
#include<mutex>

std::mutex m1, m2, m3;
using MuxGuard = std::lock_guard<std::mutex>;

class Widget
{
    public:
        void print()
        {
            std::cout << "I am a Widget" << std::endl;            
        }
};

int f1(std::shared_ptr<Widget> spw){ return 0;}
double f2(std::unique_ptr<Widget> upw){return 0.0;}
bool f3(Widget* pw){return true;}

template<typename FuncType,typename MuxType,typename PtrType>
auto lockAndCall(FuncType func, MuxType& mutex, PtrType ptr) -> decltype(func(ptr))
{
    MuxGuard g(mutex);
    return func(ptr);
}

int main()
{
    //gcc 9.1.0 C++2a
    //auto result =  lockAndCall(f1, m1, 0); //error: could not convert 'ptr' from 'int' to 'std::shared_ptr<Widget>'
    //auto result =  lockAndCall(f1, m1, NULL); // error: could not convert 'ptr' from 'long int' to 'std::shared_ptr<Widget>'
    auto result = lockAndCall(f1, m1, nullptr); //OK
}
```

Intermediate result from cppinsights.io

```
template<typename FuncType,typename MuxType,typename PtrType>
auto lockAndCall(FuncType func, MuxType& mutex, PtrType ptr) -> decltype(func(ptr))
{
    MuxGuard g(mutex);
    return func(ptr);
}

/* First instantiated from: insights.cpp:33 */
#ifdef INSIGHTS_USE_TEMPLATE
template<>
int lockAndCall<int (*)(std::shared_ptr<Widget>), std::mutex, nullptr_t>(int (*func)(std::shared_ptr<Widget>), std::mutex & mutex, nullptr_t ptr)
{
  MuxGuard g = std::lock_guard<std::mutex>(mutex);
  return func(std::shared_ptr<Widget>(ptr));
}
#endif


int main()
{
  int result = lockAndCall(f1, m1, nullptr);
}

```

Please note that a specialized version of the `lockAndCall` has been generated.
