# Prefer alias to typedef

C++11 introduced alias declaration which basically do the same thing than typedefs, except not quite exactly the same thing.

What are the differences?

1. Function pointers are easier to write. Using is just much more clear than typedef.
```cpp
typedef void(*fprt)(bool a, const string&);
using fprt=  void(*)(bool a, const string&);
```

2. alias can be template and in this case are called alias template i.e. alias having a template parameter in their signature. typedefs cannot be template. Simple as that.

```cpp
template<typename T>
using MyallocList = std::list<T, Myallocator<T>>;

//you can use as you would expect:
MyallocList<double> listdouble;

template<typename T>
struct {
	typedef std::list<T, Myallocator<T>> type;
};

//this one has to be used in a weird way.
MyallocListOLD<double>::type listdouble; 
```

3. When you use the typedef as a dependent name (what does it even mean?) you should preceed the name by `typename` keyword. A dependent name is basically a name that depends on a template parameter.  this happens all the time you are tying to use `MyallocListOLD::type` inside another template class.

> A name used in a template declaration or definition and that is dependent on a template-parameter is assumed not to name a type unless the applicable name lookup finds a type name or the name is qualified by the keyword typename. 

```cpp
template<typename U>
struct ClassUsingMyList{

typename MyallocListOLD<U>::type list; //this is a dependent name! this typedef depend on a template parameter. you need typename
};
```

If you use aliases then there is no need to use typename anywhere. there is no ambiguity when using `MyallocListOLD::type` whether `type` is a type or a variable.
Where this ambiguity comes from?
You might have a specialization of `MyallocListOLD` where the typedef is not there and `type` is not a type but a member variable!

```cpp
template<typename T>
struct MyallocListOLD{
	typedef std::list<T, Myallocator<T>> type;
};

template<>
struct MyallocListOLD<double>
{
	int type;
};

template<typename U>
struct ClassUsingMyList{
	MyallocListOLD<U>::type list; //you see why is ambiguous?
};

template<typename K, typename V>
struct ClassUsingMyList{
	map<K,V> map;
}

```
ClassUsingMyList<int>    => MyallocListOLD<U>::type = std::list<int, Myallocator<int>>
ClassUsingMyList<Widget> => MyallocListOLD<U>::type = std::list<Widget, Myallocator<Widget>>

# Type Traits : aliases or typedefs?
In template metaprogramming you often need to perform some change on the template. You have dozens of those type traits in C++11 and if you start doing TMP most likely you need to get familiar with those.

```cpp
#include <iostream>
#include <type_traits>
 
struct foo
{
    void m() { std::cout << "Non-cv\n"; }
    void m() const { std::cout << "Const\n"; }
    void m() volatile { std::cout << "Volatile\n"; }
    void m() const volatile { std::cout << "Const-volatile\n"; }
};
 
int main()
{
    foo{}.m(); //void m()
    std::add_const<foo>::type{}.m(); //void m() const
    std::add_volatile<foo>::type{}.m(); // void m() volatile
    std::add_cv<foo>::type{}.m(); // void m() const volatile
}
```
in C++11 all typetraits are still implemented using typedefs.... so you still suffer from all the problems described above. Infact this would happen wwhen using `std::remove_reference`

```cpp
template<class T>
class K{
	typename std::remove_reference<T>::type Tnoref; //if the template parameter was T& then Tnoref is a variable of Type T
};
```

In C++14 they added a alias implementation of type traits
```cpp
template<class T>
class K{
	std::remove_reference_t<T> Tnoref; 
};
```
