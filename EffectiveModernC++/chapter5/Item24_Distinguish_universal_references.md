# Item 24 - Distinguish universal references from rvalue references

Note: contains code snippets from Effective Modern C++ by Scott Meyers

A common mistake is that whenever you see reference to some type indicating `T&&` you assume that you are looking at rvalue reference.
However, that is not always the case. For instance, the following code snippets do not indicate rvalue references.

```c++
auto&& var2 = var1; //not an rvalue reference

template<typename T>
void f(T&& param); //not an rvalue reference

```

`T&&` can either point only to rvalues or it can point to either rvalue or lvalue references. `T&&` can also indicate as if they were 
lvalue references(`T&`). They can bind to const, non const, volatile or non-volatile objects. The author calls such references as universal
references or forwarding references (because we almost aplways apply *std::forward* to it - item 25).

For instance, the follwing indicate universal references

```c++
template<typename T>
void f(T&& param) //param => universal reference

auto &&var2 = var1; //var2 => universal reference

```

In both these cases, you can see that type deduction takes place. In cases where there is no type deduction, like below, you can see that
they indicate rvalue references

```c++
void f(Widget&& param); // no type deduction, param refers to rvalue references
Widget&& var1 = Widget(); //no type deduction, var1 is an rvalue reference

```

The initializer determines whether the universal reference is an rvalue or an lvalue.

```c++
template<typename T>
void f(T&& param); //universal reference

Widget w;
f(w); //lvalue, params type is Widget&

f(std::move(f)); // rvalue, params type is Widget&&

```

For a reference to be universal, type deduction must be present but that alone does not indicate that the reference is universal. For instance, in the following snippet, param is an rvalue reference

```c++
template<typename T>
void f(std::vector<T>&& param);// param=> rvalue reference 
std::vector<int> v;
f(v); // error
```

```c++
template<typename T>
void f(const T&& param); //rvalue reference
```

```c++
template<class T, class Allocator = allocator<T>>
class vector{
public:
  void push_back(T&& x);
}
```

When you instantiate a vector `std::vector<Widget> v`, then the std::vector template is as follows:

```c++
template<class Widget, class Allocator = allocator<Widget>>
class vector{
public:
  void push_back(Widget&& x);
}
```
No type deduction takes place. 


```c++
template<class T, class Allocator = allocator<T>>
class vector{
public:
  template<class... Args>
  void emplace_back(Args&&... args); // form of universal reference
}
```

```c++
auto timeFuncInvocation =  [](auto&& func, auto&&... params) // func can bind to both rvalue and lvalue references
{

}
```

In summary, if a function template is of type T&& and T is deduced, or if an object declaration is indicated using `T&&`, then
it is a universal reference. If type deduction does not occur or if the type deduction is not `type&&`, then `type&&` denotes an rvalue
reference. Universal references correspond to rvalue references if they are initialized with rvalues, lvalues if they are initialized 
with lvalues.


