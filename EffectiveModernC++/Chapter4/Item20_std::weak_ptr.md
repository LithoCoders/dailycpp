# Use `std::weak_ptr` for `std::shared_ptr`-like pointers that can dangle

`std::weak_ptr`s are like `std::shared_ptr`s but not participate in ownership of the pointed-to resource, but they are able to track if resource is destroyed, so-called, *dangling pointers*
This also means, `std::weak_ptr`s do not affect to object's reference count.

Here are some characteristics:
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
We can check for nullnes of `std::shared_ptr` but not `std::weak_ptr`
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
    
    std::cout  << "Does obj exist ? "  << wpi.expired()  << std::endl;
    spi = nullptr;
    std::cout  << "Does obj exist ? "  << wpi.expired()  << std::endl;    
    
    return 0;
}
Start

Does obj exist ? 0
Does obj exist ? 1

0

Finish
```




