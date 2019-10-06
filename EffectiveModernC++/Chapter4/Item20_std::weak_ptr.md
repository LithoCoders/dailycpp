# Use `std::weak_ptr` for `std::shared_ptr`-like pointers that can dangle

`std::weak_ptr`s are like `std::shared_ptr`s but not participate in ownership of the pointed-to resource, but they are able to track if resource is destroyed, i.e., *dangling pointers*
This also means, `std::weak_ptr`s do not affect to object's reference count.

## Some characteristics:
* `std::weak_ptr`s can be created from `std::shared_ptr`s
* it can't be dereferenced
* it can't be tested for nullness
* it can check if it dangles

```c++
#include <iostream>
#include <memory>

int main()
{ 
    std::shared_ptr<int> spi = std::make_shared<int> (1);    
    std::cout << "spi's RC: " << spi.use_count() << std::endl;
    
    std::weak_ptr<int> wpi(spi);        //does not increase RC
    std::cout << "Still, spi's RC: " << spi.use_count() << std::endl;
    
    return 0;
}
Start

spi's RC: 1
Still, spi's RC: 1

0

Finish
```
Let us try to dereference `wpi`, 

```c++
std::cout  << *wpi << std::endl;

error: no match for 'operator*' (operand type is 'std::weak_ptr<int>')
        std::cout  << *wpi << std::endl;
                      ^~~~
```
We can check for nullness of `std::shared_ptr` but not `std::weak_ptr`
```c++
std::cout  << (spi==nullptr) << std::endl;

Start

0

0

Finish
```
```c++
std::cout  << (wpi==nullptr) << std::endl;
 error: no match for 'operator==' (operand types are 'std::weak_ptr<int>' and 'std::nullptr_t')
         std::cout  << (wpi==nullptr) << std::endl;
                        ~~~^~~~~~~~~
```
Let us check if object, which our `wpi` pointing to, is destroyed with `expired()` function

```c++
#include <iostream>
#include <memory>

int main()
{ 
    std::shared_ptr<int> spi = std::make_shared<int> (1);    
    std::weak_ptr<int> wpi(spi);            
    
    std::cout  << std::boolalpha <<  "Is obj expired ? "  << wpi.expired()  << std::endl;
    spi = nullptr;     //wpw now dangles
    std::cout  << "Is obj expired ? "  << wpi.expired()  << std::endl;    
    
    return 0;
}
Start

Is obj expired ? false
Is obj expired ? true

0

Finish
```
## How to access to object pointed by a `std::weak_ptr` ?
Often the case we would like to access to object, pointed by a `std::weak_ptr`, so we need to:
* check if `std::weak_ptr` is expired ? Is object destroyed?
* dereference pointere to access to object.
But `std::weak_ptr` does not support reference. 

You need to create a `std::shared_ptr` from `std::weak_ptr` so that you can manipulate object. There are two ways to do this: using `lock()` or using `std::shared_ptr` constructor

```c++
#include <iostream>
#include <memory>

int main()
{ 
    std::shared_ptr<int> spi1 = std::make_shared<int> (1);    
    std::weak_ptr<int> wpi(spi1);            
    std::cout  << std::boolalpha <<  "Is obj expired ? "  << wpi.expired()  << std::endl;
    
    //using lock()
    std::shared_ptr<int> spi2 = wpi.lock();
    std::cout << "RC: " << spi2.use_count() << std::endl;   
    std::cout << "obj = " << *spi2 << std::endl;   
    
    //using shared_ptr ctor
    std::shared_ptr<int> spi3 (wpi);
    std::cout << "RC: " << spi3.use_count() << std::endl;   
    std::cout << "obj = " << *spi3 << std::endl;   
    
    spi3=nullptr;
    spi2=nullptr;
    spi1=nullptr;
    std::cout << wpi.expired() << std::endl;   
    std::shared_ptr<int> spi4 (wpi);
        
    return 0;
}
Start

Is obj expired ? false
RC: 2
obj = 1
RC: 3
obj = 1
true

terminate called after throwing an instance of 'std::bad_weak_ptr'
  what():  bad_weak_ptr

Aborted

Finish
```

In the above example, when `wpi` is expired (all shared pointers are null), then run-time exception is thrown, saying `std::bad_weak_ptr`

We observe that `std::shared_ptr` and `std::weak_ptr` can be created from constructor, taking the other one as argument.
```c++
std::weak_ptr<int> wpi(spi);        

std::shared_ptr<int> spi (wpi);
```







