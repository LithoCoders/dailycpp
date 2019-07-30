# Item 8 - Prefer nullptr to 0 and NULL

Note: Contains code snippet from Effective Modern C++ by Scott Meyers


The author suggests to use `nullptr` in place of NULL or 0 to prevent the follwing issues:

1. To prevent overloading on pointer to integer types
2. To Prevent ambiguity

```c++
NULL -> either int or long depending on implementation
0 -> int
nullptr -> std::nullptr_t -> implicitly converts to all raw pointer types -> not an integral type
std::nullptr_t -> nullptr -> circular definition
```

```c++

void f(int);
void f(bool);
void f(void*);


f(0); //calls f(int)

f(NULL); //might not compile, but may call f(int)

f(nullptr); //c++11, calls f(void*)

auto result = findRecord( /* arguments */ );

if(result == 0){} // unclear what findRecord returns

if(result == nullptr){} //result must be a ptr type

```

```c++


#include<iostream>
#include<memory>
#include<mutex>

std::mutex m1, m2, m3;
using MuxGuard = std::lock_guard<std::mutex>;

class Widget
{
    
};

// code duplication - mutex lock
int f1(std::shared_ptr<Widget> spw)
{
    MuxGuard g(m1); // lock mutex for f1
    return 0;
}
double f2(std::unique_ptr<Widget> upw)
{
    MuxGuard g(m2); // lock mutex for f1
    return 0.0;
}

bool f3(Widget* pw)
{
    MuxGuard g(m3); // lock mutex for f1
    return 0;
}


int main()
{
  auto result = f1(NULL); // all three work
  auto result2 = f2(0);
  auto result3 = f3(0);
}

```

The above implementation from the compiler is NOK because of code duplication and usage of NULL and 0. Why does it work? Because the
compiler does it's job for you ;)

Intermediate compiler output from cppinsights.io for the above impl(C++11):

```
int main()
{
  int result = f1(std::shared_ptr<Widget>(std::shared_ptr<Widget>(NULL)));
  double result2 = f2(std::unique_ptr<Widget, std::default_delete<Widget> >(std::unique_ptr<Widget, std::default_delete<Widget> >(0)));
  bool result3 = f3(0);
}
```
Note that the NULL and 0 are casted to the respective `shared_ptr` and `unique_ptr` types.

```c++

//template impl

#include<iostream>
#include<memory>
#include<mutex>

std::mutex m1, m2, m3;
using MuxGuard = std::lock_guard<std::mutex>;

class Widget
{
    
};

int f1(std::shared_ptr<Widget> spw){ return 0;}
double f2(std::unique_ptr<Widget> upw){return 0.0;}
bool f3(Widget* pw){return 0;}

template<typename FuncType,typename MuxType,typename PtrType>
auto lockAndCall(FuncType func,
MuxType& mutex,
PtrType ptr) -> decltype(func(ptr))
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
auto lockAndCall(FuncType func,
MuxType& mutex,
PtrType ptr) -> decltype(func(ptr))
{
MuxGuard g(mutex);
return func(ptr);
}

/* First instantiated from: insights.cpp:34 */
#ifdef INSIGHTS_USE_TEMPLATE
template<>
int lockAndCall<int (*)(std::shared_ptr<Widget>), std::mutex, nullptr_t>(int (*func)(std::shared_ptr<Widget>), std::mutex & mutex, nullptr_t ptr)
{
  MuxGuard g = std::lock_guard<std::mutex>(mutex);
  return func(std::shared_ptr<Widget>(std::shared_ptr<Widget>(ptr)));
}
#endif


int main()
{
  int result = lockAndCall(f1, m1, nullptr);
}

```

Please note that a specialized version of the `lockAndCall` has been generated.
