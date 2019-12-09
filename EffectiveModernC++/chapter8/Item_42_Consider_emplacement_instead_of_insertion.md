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

int main()
{
    std::vector<std::string> vs;
    vs.emplace_back("abc");     //string literal
    vs.emplace_back(50, 'x');
    return 0;
}
```

Heuristic when to use emplace_back()
