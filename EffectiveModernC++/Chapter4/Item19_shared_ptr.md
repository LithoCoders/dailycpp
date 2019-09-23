# std::shared_ptr usage

With `std::shared_ptr`, clients need not to concern with managing life-time of *pointed-to* objects (as garbage collector). Beside, the timing of the objects' destruction is deterministic.

* Using a shared pointer is like a normal raw pointer
* *Reference count* is how a shared pointer knows if it is the last one pointing to an object.

```c++
#include <memory>
#include <iostream>

int main()
{
    std::shared_ptr<int> sp1 = std::make_shared<int>(1);
    std::shared_ptr<int> sp2 = std::make_shared<int>(2);
    std::cout << "use_count = " << sp1.use_count() << std::endl;
    sp2 = sp1;
    
    //dereference a shared pointer like a raw pointer
    std::cout << "*sp1= " << *sp1 << ", *sp2= " << *sp2 << std::endl;
    *sp1 += 1;
    std::cout << "*sp1= " << *sp1 << ", *sp2= " << *sp2 << std::endl;   
    
    //check number of pointers point to the same object
    if (sp1.use_count() == sp2.use_count())
        std::cout << "use_count = " << sp2.use_count() << std::endl;
    else
        std::cout << "something wrong" << std::endl;  
    
    return 0;
}
Start

use_count = 1
*sp1= 1, *sp2= 1
*sp1= 2, *sp2= 2
use_count = 2

0

Finish
```
**Reference count**
* Constructor increments *reference count*
* Destructor decrements *reference count*
* Copy assignment (as example above) also decrements *reference count* to object `int` of value 2 and increments *reference count* of object `int` of value 1.
* When `shared_ptr` see *reference count* is `0` after decrement, it destroy resource.
* Increment and decrement of a *reference count* must be atomic
* For `std::shared_ptr`: move constructors are faster than copy constructors; move assignments are faster than copy assignments. Because move operators seem not care about reference counts? So, "move when moveable, copy when otherwise"

# `shared_ptr` vs. `unique_ptr`
1. `shared_ptr`s are twice the size of a raw pointer because it stores
* a raw pointer to resource
* a raw pointer to reference count (but actually later we see it is a block of memory, called *control block*)

2. `shared_ptr`s are more flexible with custom deleter, compared to `unique_ptr`s.
This is because share pointers need defining custom deleter as input of cosntructor while unique pointers need to define custom deleter in template parameter.

```c++
#include <memory>
#include <vector>
#include <iostream>

struct Widget
{
    int a;
};

void makeLogEntry(Widget* pw)
{
    std::cout <<__PRETTY_FUNCTION__<< pw->a << std::endl;
}

void makeSpecialLogEntry(Widget *pw)
{
    std::cout << __PRETTY_FUNCTION__ << pw->a << std::endl;
}

auto customDeleter = [](Widget* pw)            //custom deleter writes log before destroying resource
                      {
                        makeLogEntry(pw);
                        delete pw;
                      };

auto customSpecialDeleter = [](Widget* pw)    //custom deleter with special logging
                      {
                        makeSpecialLogEntry(pw);
                        delete pw;
                      };
int main()
{
    std::unique_ptr<Widget, decltype(customDeleter)> 
                                  p(new Widget, customDeleter); // define type of deleter in template
    
    std::shared_ptr<Widget> sp1(new Widget, customDeleter);     // define type of custom deleter in ctor
    std::shared_ptr<Widget> sp2(new Widget, customSpecialDeleter);
    
    std::vector<std::shared_ptr<Widget>> vsp;  // vector of shared pointers not care about deleters
    vsp.push_back(sp1);
    vsp.push_back(sp2);
    return 0;
}

Start

void makeSpecialLogEntry(Widget*)0
void makeLogEntry(Widget*)0
void makeLogEntry(Widget*)0

0

Finish
```
As a result, we can have 
* a vector of shared pointers with different deleters. 
* Shared pointers can be reassigned, reset its deleter. 
* Passing a share pointer to a function, with different deleters.
