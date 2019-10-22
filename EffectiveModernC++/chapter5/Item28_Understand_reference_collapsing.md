# Revisit universal reference
```c++
template<typename T>
void func(T&& param)
{}

struct Widget{};

Widget makeWidget()
{
    Widget w;
    return w;
}
int main()
{
    Widget w;         //lvalue
    func(w);          //T deduced to Widget&
    
    func(makeWidget); //T deduced to Widget (non-reference)
    
    return 0;
}
```
Deduction rules as in item 1:
* When `func` takes lvalue as an argument, `T` deduced to *lvalue reference*
* When `func` takes rvalue as an argument, `T` deduced to *non-reference*

# References to references are illegal in C++

```c++
int main()
{
    int x = 1;
    auto& y = x;  // OK, y is a reference to integer, int&
    
    auto& &z = y; //illegal, reference to a reference, i.e., auto&
    int& &w = y;  //illegal, reference to a reference to int, i.e., int&
    
    return 0;
}
error: cannot declare reference to 'auto&', which is not a typedef or a template type argument
        auto& &z = y;
               ^
error: cannot declare reference to 'int&', which is not a typedef or a template type argument
        int& &w = y; 
              ^
```

# Reference collapsing

You cannot declare references to references but compiler may produce them. And when this happens, *reference collapsing* happens.
There are two types of references: 
* lvalue reference
* rvalue reference

Thus, there are four difference combinations:
1. lvalue reference  ---to---> lvalue reference,i.e., `Widget& &w`
2. lvalue reference  ---to---> rvalue reference,i.e., `Widget&& &w`
3. rvalue reference  ---to---> lvalue reference,i.e., `Widget& &&w`
4. rvalue reference  ---to---> rvalue reference,i.e., `Widget&& &&w`

Rules quoted from the book:
"If either reference is an lvalue reference, the result is an lvalue reference. Otherwise (i.e., if both are rvalue references) the result is an rvalue reference"

In short, if both are rvalue reference, result is rvalue reference (4.). The rests, lvalue reference (1., 2., 3.)

Applying reference collapsing:
1. `Widget& &w` yields `Widget& w`
2. `Widget&& &w` yields `Widget& w`
3. `Widget& &&w` yields `Widget& w`
4. `Widget&& &&w` yields `Widget&& w`

Consider example:
```c++
    func(w);
    //func(Widget& && param); Replacing T with Widget& in function template
    //rvalue reference to lvalue reference, result is a lvalue reference
    //func(Widget& param);
    
    func(makeWidget); 
    //func(Widget&& && param);
    //rvalue reference to rvalue reference, result is a rvalue reference
    //func(Widget&& param);
```
# `std::forward<T>(param)`
*Reference collapsing* is the key to understand how `std::forward` works.
The following code is relative to actual implementation (written by the author)

```c++
template<typename T>
func(T&& fParam)
{
  someFunc(std::forward<T>(fParam));
}

template<typename T>
T&& forward(typename remove_reference<T>::type& param) //relatively implementation
{
  return static_cast<T&&>(param);
}
```
Let us see how `forward` is instantiated:

Case 1: `T` is deduced to `Widget&`:

```c++
Widget& && forward(typename remove_reference<Widget&>::type& param)
{
  return static_cast<Widget& &&>(param);
}
```
`remove_reference<Widget&>::type` yields `Widget`
```c++
Widget& && forward(Widget& param)
{
  return static_cast<Widget& &&>(param);
}
```
after reference collapsing
```c++
Widget& forward(Widget& param)
{
  return static_cast<Widget&>(param); // cast does nothing 'cause param is already a lvalue reference
}
```
So, when `T` is `Widget&`, we have `func(Widget& fParam)`, and `forward` takes `Widget&` and returns `Widget&`

Quote from the book: "By definition, lvalue references are lvalues, so passing an lvalue to std::forward causes an lvalue to be returned."

Case 2: `T` is deduced to `Widget`:
```c++
Widget&& forward(typename remove_reference<Widget>::type& param)
{
  return static_cast<Widget&&>(param);
}
```
`remove_reference<Widget>::type&` yields `Widget&`
```c++
Widget&& forward(Widget& param)
{
  return static_cast<Widget&&>(param);
}
```
"rvalue argument passed to f will be forwarded to someFunc as an rvalue". Argument is eligible to be moved.

Only in C++14:
```c++
template<typename T>
T&& forward(remove_reference_t<T>& param)
{
    return static_cast<T&&>(param);
}
```

# auto
Auto deduction works the same like function template deduction, Item 2. As reference collapsing happens in template deduction, it also happens in auto deduction.

```c++
template<typename T>
void func(T&& param)
{}

struct Widget{};

Widget makeWidget()
{
    Widget w;
    return w;
}
int main()
{
    Widget w;         //lvalue
    auto&& w1 = w;
    
    auto&& w2 = makeWidget();
    
    return 0;
}
```
First case, `w` is an lvalue reference, thus `Widget&`. Let us plug this to `w1` declaration:
```c++
    Widget& && w1 = w;
    //after reference collapsing
    Widget& w1 = w;
```
Second case, `makeWidget()` returns object of `Widget`. Let us plug this to `w2` declaration:
```c++
    Widget && w2 = makeWidget();
    //No reference collapsing
    Widget&& w2 = makeWidget();
```

# typedef and allias declarations
Item 9.
```c++
template<typename T>
class Widget
{
    public: 
        typedef T&& RvalueRefToT;
};

//Call side:
Widget<int&> w;
```
Template instantiation:

```c++
    public: 
        typedef int& && RvalueRefToT;
```
Reference collapsing:
```c++
    public: 
        typedef int& RvalueRefToT; //Wait, naming is not correct
                                   // it should have been: lvalueRefToT
```

# decltype
No example. Helaas!

# In summary:
Reference collapsing happens in four cases: template deduction, auto deduction, typedef/allias, and decltype.
