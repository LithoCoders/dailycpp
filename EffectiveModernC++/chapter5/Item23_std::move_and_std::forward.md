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


