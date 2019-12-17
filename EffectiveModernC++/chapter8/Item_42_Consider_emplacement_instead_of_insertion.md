# Item 42: Consider emplacement instead of insertion

Function `push_back()` of `std::vector` is overloaded taking both lvalue reference and rvalue reference
```c++
template<class T, class Allocator = allocator<T>>
class vector
{
  public:
    void push_back (const value_type& val);
    void push_back (value_type&& val);
};
```
In the calling side:

```c++
#include <vector>
#include <string>

int main()
{
    std::vector<std::string> vs;
    vs.push_back("abc");     //string literal
    return 0;
}
```

There is no `push_back()` function taking string literal as argument, so that the equivalent code is
```c++
    vs.push_back(std::string("abc"));
```
The following steps are performed:
1. A *temporary* string object is created, this is an rvalue.
2. `push_back()` taking rvalue reference is called, thus, move ctor to create an object at the end of vector.
3. A *temporary* string object is destroyed.

We see that both two steps: first and last can be skipped for performance. We can use `emplace_back()` to perfect forward argument to move ctor to create an object at the end of vector.
Because of perfect forwarding, we can forward any argument to create a string, e.g., `std::string(50, 'x')`.
```c++
#include <vector>
#include <string>
#include <initializer_list>
#include <iostream>

int main()
{
    std::vector<std::string> vs;
    vs.emplace_back("abc");     //string literal
    vs.emplace_back(50, 'x');   //because of ctor: string (size_t n, char c);
    std::initializer_list<char> il = {'a', 'b', 'c'};
    vs.emplace_back(il);        //because of ctor: string (initializer_list<char> il);
    for(auto i : vs) {std::cout << i << std::endl;}
    return 0;
}
/*
abc
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
abc
*/
```
Function `emplace_back()` of `std::vector` takes variadic rvalue references

```c++
template <class... Args>
  void emplace_back (Args&&... args);
```

The above code on C++ Insight looks like:
```c++
  vs.emplace_back<char const (&)[4]>("abc");
  vs.emplace_back<int, char>(50, 'x');
  std::initializer_list<char> il = std::initializer_list<char>{'a', 'b', 'c'};
  vs.emplace_back<std::initializer_list<char> &>(il);
```

Function `emplace_back()` is more flexible compared to `push_back()` because emplacement functions take *constructor arguments for objects to be inserted*, whereas insertion functions take *objects to be inserted.*

Emplacement functions are for example: `list::emplace_back()`, `deque::emplace_front()`, `forward_list::emplace_after()`
Insertion functions are: `vector::push_back()`, `list::push_front()`

The following code shows emplacement and insertion functions have the same effect on `vs`, namely, adding one element to the end of vector.
```c++
#include <string>
#include <vector>
int main()
{
    std::vector<std::string>  vs;
  
  	std::string s("abc");
	  vs.push_back(s);
	  vs.emplace_back(s);  //lvalue reference so no temporary string object created
}

//C++ Insight

#include <string>
#include <vector>
int main()
{
  std::vector<std::string> vs = 
    std::vector<std::basic_string<char, std::char_traits<char>, std::allocator<char> >, 
                                  std::allocator<std::basic_string<char, std::char_traits<char>, std::allocator<char> > > >();
  
  std::string s = std::basic_string<char, std::char_traits<char>, std::allocator<char> >("abc", std::allocator<char>());
  
  vs.push_back(s);
  vs.emplace_back<std::basic_string<char, std::char_traits<char>, std::allocator<char> > &>(s);
}



```
Emplacement functions can do what insertion functions do, sometimes even more efficient. But not all the cases, here is some heuristics when to use emplacement functions:
1. "The value being added is constructed into the container, not assigned." 
2. "The argument type(s) being passed differ from the type held by the container"
3. "The container is unlikely to reject the new value as a duplicate."

Consider the following piece of code satisfying the above conditions:

```c++
vs.emplace_back("xyzzy"); // construct new value at end of
// container; don't pass the type in
// container; don't use container
// rejecting duplicates
vs.emplace_back(50, 'x'); // ditto
```

Also the following cases need to be considered:

```c++
std::list<std::shared_ptr<Widget>> ptrs;
void killWidget(Widget* pWidget);


ptrs.push_back(std::shared_ptr<Widget>(new Widget, killWidget)); // Either way could be used
ptrs.push_back({ new Widget, killWidget });
```

If you have a `std::shared_ptr` that should use a custom deleter, you cannot use `std::make_shared`(see item 21). A temporary `std::shared_ptr` would need to be constructed before calling `push_back`. The construction of `std::shared_ptr` is something `emplace_back` would avoid. However the advantage of this is that when an exception is thrown during the allocation of the node to hold this object, there is no problem as the memory will be released. In the case of `emplace_back`:

```c++
ptrs.emplace_back(new Widget, killWidget)
```
 In this case the `new Widget` is perfect-forwarded to the place where the memory for the new node is to be allocated. However, if an
 exception is thrown in this case, it leads to a memory leak.
